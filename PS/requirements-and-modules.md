# Core System Modules, Functional Requirements & Conceptual Data Model

**Back to [[locus]]**

---

## 1. System Modules Overview

Locus is structured into 10 decoupled functional modules operating within the desktop application:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                            LOCUS SYSTEM MODULES                             │
│                                                                             │
│  [M01] Evidence Ingestion & Cryptographic Hashing Engine                    │
│  [M02] Device Identification & Storage Layout Scanner                       │
│  [M03] Storage & Filesystem Analysis Engine                                 │
│  [M04] Modular Vendor Adapter Pipeline                                      │
│  [M05] Video Recovery & Sector Carving Engine                               │
│  [M06] Media Validation Engine                                              │
│  [M07] Time Engine & Multi-Camera Timeline Normalizer                       │
│  [M08] Secondary AI Analytics Engine (ONNX Runtime)                         │
│  [M09] Custody, Provenance & Transaction Audit Logger                       │
│  [M10] Forensic Evidence Report Generator                                   │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Detailed Functional Requirements per Module

### Module 01: Evidence Ingestion & Cryptographic Hashing
- **REQ-M01-1:** Ingest pre-acquired raw disk images (`.dd`, `.raw`, `.img`) via desktop UI file dialogs.
- **REQ-M01-2:** Open file handles using strict software read-only flags (`rb` mode).
- **REQ-M01-3:** Calculate baseline `SHA-256` and `MD5` cryptographic hashes using 64 KB streaming buffers upon registration.
- **REQ-M01-4:** Record file size, baseline hashes, ingestion timestamp, and operator identity in SQLite database.

### Module 02: Device Identification & Storage Layout Scanner
- **REQ-M02-1:** Execute sector-aligned magic header scanner across disk image boundaries.
- **REQ-M02-2:** Match signatures against vendor profile patterns (e.g., Dahua `DHAV`, Hikvision `HKFS`).
- **REQ-M02-3:** Assign vendor identification confidence level (`KNOWN`, `LIKELY`, `UNKNOWN`, `UNSUPPORTED`, `AMBIGUOUS`).
- **REQ-M02-4:** Fallback to `UNKNOWN` classification when headers are malformed or unverified.

### Module 03: Storage & Filesystem Analysis Engine
- **REQ-M03-1:** Parse Master Boot Record (MBR) and GUID Partition Table (GPT) structures at sector 0 and sector 1.
- **REQ-M03-2:** Map custom partition layouts and unallocated sector spaces.
- **REQ-M03-3:** Identify master index tables where surviving filesystem entries reside.

### Module 04: Modular Vendor Adapter Pipeline
- **REQ-M04-1:** Define standardized adapter interface (`IVendorAdapter`).
- **REQ-M04-2:** Implement `DahuaDHAVAdapter` for index parsing and sector carving fallback.
- **REQ-M04-3:** Implement `HikvisionHKFSAdapter` for HKFS block structures.
- **REQ-M04-4:** Mark adapter validation status explicitly (`PLANNED`, `RESEARCHING`, `PARTIALLY VALIDATED`, `VALIDATED`).

### Module 05: Video Recovery & Sector Carving Engine
- **REQ-M05-1:** Parse surviving filesystem index entries to extract intact recordings.
- **REQ-M05-2:** Execute sector-aligned carving across unallocated sectors when index entries are corrupted or overwritten.
- **REQ-M05-3:** Assign recovery status: `RECOVERED`, `PARTIAL`, `FRAGMENTED`, `CORRUPTED`, `UNRECOVERABLE`, `UNSUPPORTED`.
- **REQ-M05-4:** Never claim recovery of physically overwritten sectors.

### Module 06: Media Validation Engine
- **REQ-M06-1:** Parse recovered container structures (AVContainer, GOP headers, NAL units).
- **REQ-M06-2:** Test frame decodability via PyAV/FFmpeg bindings.
- **REQ-M06-3:** Assign media validation flags (`VALID`, `PARTIAL`, `CORRUPTED`, `UNSUPPORTED`).

### Module 07: Time Engine & Multi-Camera Timeline Normalizer
- **REQ-M07-1:** Extract and preserve raw in-stream timestamps from frame headers.
- **REQ-M07-2:** Support investigator clock-offset calibration to correct DVR time drift.
- **REQ-M07-3:** Calculate normalized master UTC timestamp timeline for cross-camera correlation.

### Module 08: Secondary AI Analytics Engine (ONNX Runtime)
- **REQ-M08-1:** Sample validated video clips at configurable frame intervals (e.g., 1 fps).
- **REQ-M08-2:** Execute local CPU ONNX Runtime inference using YOLOv8 (`person`, `vehicle` detection).
- **REQ-M08-3:** Store detection bounding boxes, confidence scores, and timestamps as secondary analytical tags.
- **REQ-M08-4:** Require investigator review status (`Verified`, `Rejected`, `Unreviewed`).

### Module 09: Custody, Provenance & Transaction Audit Logger
- **REQ-M09-1:** Log all system operations, configuration edits, and export requests into an append-oriented, tamper-evident `audit_log` table.
- **REQ-M09-2:** Maintain exact provenance trace for every derived clip (Source Image ID, Offset, Length, Adapter Version, SHA-256 Hash).

### Module 10: Forensic Evidence Report Generator
- **REQ-M10-1:** Export structured HTML and PDF Forensic Evidence Reports.
- **REQ-M10-2:** Include evidence baseline hashes, artifact hash parity tables, provenance details, human-reviewed AI findings, and uncertainty disclaimers.

---

## 3. Conceptual Data Model & Entity Relationships

The local SQLite database (`case_meta.db`) models evidence metadata without storing raw video payloads:

```
┌──────────────┐       1:N       ┌──────────────────┐
│     Case     ├─────────────────┤  EvidenceSource  │
└──────┬───────┘                 └────────┬─────────┘
       │                                  │ 1:N
       │ 1:N                              ▼
       │                         ┌──────────────────┐
       │                         │ DeviceIdentification│
       │                         └────────┬─────────┘
       │                                  │ 1:1
       │                                  ▼
       │                         ┌──────────────────┐
       │                         │  SelectedAdapter │
       │                         └────────┬─────────┘
       │                                  │ 1:N
       │                                  ▼
       │                         ┌──────────────────┐
       │                         │ RawSectorBlock   │
       │                         └────────┬─────────┘
       │                                  │ 1:N
       │                                  ▼
       │                         ┌──────────────────┐
       │                         │ DerivedArtifact  │
       │                         └──┬────────────┬──┘
       │                            │ 1:N        │ 1:N
       ▼                            ▼            ▼
┌──────────────┐           ┌──────────────┐ ┌──────────────┐
│  AuditEvent  │           │ Provenance   │ │ AIDetection  │
└──────────────┘           └──────────────┘ └──────────────┘
```

---

## 4. Non-Functional Requirements & Forensic Safeguards

1. **Design Alignment with ISO/IEC 27037:** System architecture aligns with digital evidence handling principles (integrity preservation, complete documentation, reproducibility).
2. **Resource Boundness:** Hash calculation uses fixed-size 64 KB streaming buffers; sector/header signature scanning uses 512-byte sector-aligned reads. Combined, these cap backend memory utilization under 2.5 GB peak RAM.
3. **Fault Isolation & Graceful Recovery:** Corrupted sector blocks or invalid frame headers trigger explicit `CORRUPTED` flags without terminating background worker threads.
4. **Local Execution Security:** All REST endpoints and WebSocket connections bind strictly to `127.0.0.1`. Network streaming is disabled.
