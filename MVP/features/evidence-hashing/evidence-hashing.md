# Feature 08: Cryptographic Hash Verification & Provenance Sidecar Export

**Back to [[MVP/MVP|MVP]]**

---

## 1. Cryptographic Integrity & Chain of Custody Model

Locus clearly distinguishes four related forensic concepts:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                            FORENSIC AUDIT MATRIX                            │
│                                                                             │
│  [HASHING]      Mathematical byte-level fingerprint (SHA-256 / MD5).        │
│  [PROVENANCE]   Lineage tracing derived clips to source byte offsets.       │
│  [CHAIN OF CUSTODY] Documentation of physical/digital handling & custody.   │
│  [AUDIT LOG]    System activity and operation transaction record.           │
└─────────────────────────────────────────────────────────────────────────────┘
```

> [!IMPORTANT]
> **Hashes Alone Are NOT Chain of Custody:**  
> Computing a SHA-256 hash proves that a specific file has not changed in bytes. It does **not** prove who handled the disk, how the image was stored, or whether the upstream acquisition was conducted properly. Locus combines cryptographic hashing with transaction audit logs and provenance sidecars to support forensic examination.

---

## 2. Dual Cryptographic Hashing Protocol

Locus calculates dual hashes (`SHA-256` & `MD5`) using 64 KB streaming block reads:

1. **Ingestion Baseline:** Hash computed on raw `.dd` image file upon registration (`evidence_sources` table).
2. **Pre-Export Verification:** Source image hash re-calculated prior to artifact extraction to detect bit-rot or external file modification.
3. **Artifact Export Fingerprint:** Hash computed on derived `.mp4` media clip immediately upon creation (`derived_artifacts` table).

---

## 3. Cryptographic Provenance Sidecar (`.sync.json`)

Every exported media clip is packaged with a JSON provenance sidecar document:

```json
{
  "artifact_provenance": {
    "locus_version": "1.0.0",
    "export_timestamp_utc": "2026-08-25T14:30:00Z",
    "case_id": "CASE-2026-882",
    "investigator": "Officer Smith (ID: 409)"
  },
  "source_evidence": {
    "evidence_id": "EVD-2026-001",
    "file_name": "dahua_nvr_dump.dd",
    "baseline_sha256": "8c2a5f4b9d1e38a7c6e0f2b4d6a8c1e3f5b7d9a0c2e4f6a8b1d3f5a7c9e1b3d5",
    "baseline_md5": "c4f8a1e2d3b5c7a9e0f1b3d5a7c9e1b3",
    "pre_export_verification": "VERIFIED_MATCH"
  },
  "extraction_lineage": {
    "channel_id": 2,
    "source_start_offset_bytes": 104857600,
    "source_length_bytes": 14857600,
    "adapter_used": "DahuaDHAVAdapter v1.2.0",
    "recovery_status": "RECOVERED",
    "gop_aligned": true,
    "transcoding": "NONE_STREAM_COPY_ONLY"
  },
  "derived_artifact": {
    "file_name": "clip_ch02_001.mp4",
    "artifact_sha256": "4f8b91a0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a08d9e2f4a6b8c0d2e4",
    "artifact_md5": "a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6"
  }
}
```
