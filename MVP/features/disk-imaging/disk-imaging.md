# Feature 01: Pre-Acquired Forensic Image Ingestion & Baseline Hashing

**Back to [[MVP/MVP|MVP]]**

---

## 1. Primary Objective & Ingestion Boundary

The primary input to the **Locus MVP** is a **pre-acquired forensic disk image** (`.dd`, `.raw`, `.img`).

Locus is designed as a post-acquisition forensic analysis engine. Physical drive acquisition (connecting raw mechanical hard drives via hardware write-blockers) is an upstream forensic process. Ingesting pre-acquired disk images guarantees that analysis operates strictly on read-only file representations.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          UPSTREAM FORENSIC PROCESS                          │
│  Physical DVR Hard Drive ──► Hardware Write-Blocker ──► Image File (.dd)   │
└──────────────────────────────────────┬──────────────────────────────────────┘
                                       │
                                       ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                            LOCUS MVP INGESTION                              │
│  - Select Forensic Image (.dd, .raw, .img)                                  │
│  - Open File Handle in Software Read-Only Mode ('rb')                       │
│  - Stream 64 KB Chunks to Calculate Baseline SHA-256 & MD5 Hashes           │
│  - Register Evidence Metadata Record in SQLite Database                     │
└─────────────────────────────────────────────────────────────────────────────┘
```

> [!IMPORTANT]
> **Software Read-Only Analysis vs. Hardware Write Protection:**  
> Opening an evidence image read-only in software prevents application-level write calls. It does **not** replace a hardware write-blocker during physical drive acquisition. Locus explicitly documents this distinction in all evidence logs.

---

## 2. Cryptographic Baseline Verification

### Dual Hashing Strategy (`SHA-256` & `MD5`)
Upon evidence registration, Locus immediately computes baseline cryptographic hashes before any parsing or analysis begins:
- **`SHA-256`:** Primary 256-bit cryptographic digest ensuring high collision resistance.
- **`MD5`:** Secondary 128-bit digest retained for cross-referencing legacy evidence logs and compatibility.

### Streaming Block Hashing
To process multi-terabyte raw disk images without memory exhaustion, Locus implements a 64 KB block streaming reader:

```text
Disk Image (.dd) ──► 64 KB Chunk ──► hashlib.sha256().update()
                                ──► hashlib.md5().update()
```

---

## 3. Evidence Registration Metadata Schema (`evidence_sources`)

| Column Name | Data Type | Sample Value | Description |
| :--- | :--- | :--- | :--- |
| `id` | `TEXT` (PK) | `"EVD-2026-001"` | Unique evidence record identifier |
| `case_id` | `TEXT` (FK) | `"CASE-2026-882"`| Parent investigation case ID |
| `file_name` | `TEXT` | `"dahua_nvr_dump.dd"` | Original filename of evidence image |
| `file_path` | `TEXT` | `"/cases/CASE-882/dahua_nvr_dump.dd"` | Absolute local file path |
| `file_size_bytes` | `INTEGER` | `500107862016` | Exact image byte length |
| `sha256_hash` | `TEXT` | `"8c2a5f4b9d1e38a7c6e0f2b4..."` | Master baseline SHA-256 digest |
| `md5_hash` | `TEXT` | `"c4f8a1e2d3b5c7a9e0f1b3d5..."` | Master baseline MD5 digest |
| `access_mode` | `TEXT` | `"READ_ONLY_SOFTWARE"` | Explicit file access mode flag |
| `ingested_at` | `DATETIME` | `"2026-08-25T14:10:00Z"` | ISO-8601 UTC ingestion timestamp |

---

## 4. API Endpoints & Request Flow

- **Endpoint:** `POST /api/evidence/ingest`
- **Sample Ingestion Request:**
  ```json
  {
    "case_id": "CASE-2026-882",
    "file_path": "/cases/CASE-882/dahua_nvr_dump.dd",
    "investigator_id": "INV-409"
  }
  ```
- **Sample Ingestion Response:**
  ```json
  {
    "status": "INGESTED",
    "evidence_id": "EVD-2026-001",
    "file_size_bytes": 500107862016,
    "sha256": "8c2a5f4b9d1e38a7c6e0f2b4d6a8c1e3f5b7d9a0c2e4f6a8b1d3f5a7c9e1b3d5",
    "md5": "c4f8a1e2d3b5c7a9e0f1b3d5a7c9e1b3",
    "access_mode": "READ_ONLY_SOFTWARE",
    "ingested_at": "2026-08-25T14:10:00Z"
  }
  ```
