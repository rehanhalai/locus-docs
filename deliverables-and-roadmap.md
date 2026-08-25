# Detailed Deliverables, Validation Roadmap & Hackathon Demo Workflow

**Back to [[locus]]**

---

## 1. Project Roadmap & Key Milestones

### Phase 1: Foundation & Forensic Core Architecture (Aug 23 – Aug 30)
- [x] **Monorepo Architecture Setup:** Configure Electron + React + FastAPI monorepo environment.
- [x] **Database Schema & Data Model:** Implement SQLite transactional schema (`cases`, `evidence_sources`, `adapters`, `recordings`, `fragments`, `provenance`, `audit_log`, `ai_findings`).
- [x] **Evidence Ingestion Engine:** Build read-only image file handler (`.dd`, `.raw`, `.img`) with chunked 64 KB SHA-256 and MD5 hash computation.
- [x] **Ground-Truth Dataset Preparation:** Prepare controlled laboratory test datasets with known Dahua DHAV and Hikvision HKFS disk images containing known intact, deleted, and overwritten sector ranges.

### Phase 2: Vendor Adapters, Recovery & Media Validation (Aug 31 – Sept 6)
- [ ] **Device & Storage Layout Detection:** Build sector-aligned magic header scanner across disk image boundaries; implement confidence scoring engine (`KNOWN`, `LIKELY`, `UNKNOWN`, `UNSUPPORTED`, `AMBIGUOUS`).
- [ ] **Dahua Adapter (`DahuaDHAVAdapter`):** Implement DHAV index parsing and sector carving fallback logic.
- [ ] **Hikvision Adapter (`HikvisionHKFSAdapter`):** Implement HKFS partition table parsing and raw block recovery.
- [ ] **Media Validation Engine:** Demux container structures, initialize PyAV decoder, verify stream continuity, and flag corrupt payloads (`VALID`, `PARTIAL`, `CORRUPTED`, `UNSUPPORTED`).
- [ ] **Raw & UTC Time Normalizer:** Preserve in-stream raw timestamps; calculate clock drift offsets; generate master UTC timeline.

### Phase 3: Secondary AI Triage, Provenance & Forensic Reporting (Sept 7 – Sept 11)
- [ ] **Secondary AI Engine (ONNX Runtime):** Integrate YOLOv8 ONNX model for 1 fps frame sampling; index candidate object (`person`, `vehicle`) detections.
- [ ] **Human-in-the-Loop Review UI:** React interface allowing investigators to tag AI detections as `Verified`, `Rejected`, or `Unreviewed`.
- [ ] **Provenance & Audit Logging:** Record byte offsets, sector lengths, adapter versions, and hash parity for all derived `.mp4` clips.
- [ ] **Forensic Evidence Report Generator:** Generate audit-compliant PDF/HTML reports with cryptographic hash parity tables, provenance traces, and uncertainty disclaimers.

### Phase 4: Benchmarking, Failure Testing & Hackathon Demo Prep (Sept 12 – Sept 20)
- [ ] **Ground-Truth Benchmark Suite:** Measure parser accuracy, recovery percentages, false positive rates, and processing throughput on controlled datasets.
- [ ] **Failure & Uncertainty Testing:** Verify that corrupt sectors, malformed headers, and unknown layouts gracefully return `UNKNOWN` or `CORRUPTED` without crashing or inventing data.
- [ ] **Cross-Platform Executable Packaging:** Package standalone `.exe` (Windows) and `.AppImage` (Linux) bundles.

---

## 2. National-Level Hackathon Demonstration Workflow

The hackathon demonstration is structured to show a judge-friendly flow that highlights **forensic rigor, explicit uncertainty handling, and transparent provenance**:

```
[Demo Step 1: Ingestion & Baseline Cryptographic Hashing]
- Select pre-acquired raw disk dump (sample_dahua.dd).
- Register evidence record (Evidence ID: EVD-2026-001).
- System computes baseline SHA-256 and MD5 hashes via chunked read.
- UI highlights software Read-Only safeguard badge.

[Demo Step 2: Storage Layout & Device Identification]
- Execute sector-aligned signature scanner across first 100 MB.
- System detects "DHAV" magic headers at Sector Offset 2048.
- Display Candidate Identification: Vendor = Dahua, Model = NVR4xxx, Confidence = LIKELY.
- System selects DahuaDHAVAdapter (Status: PARTIALLY VALIDATED).

[Demo Step 3: Stream Recovery & Sector Carving]
- DahuaDHAVAdapter reads DHAV index table and sector frame boundaries.
- Recover 12 intact recordings and 3 surviving deleted fragments.
- Display Recovery Status: 12 RECOVERED, 2 PARTIAL, 1 CORRUPTED.
- Demonstrate explicit uncertainty: Corrupted sector range 0x08F000-0x091000 logged as CORRUPTED payload (not hidden or force-played).

[Demo Step 4: Stream Validation & Remuxing]
- Media Validation Engine demuxes recovered frame blocks.
- PyAV remuxes valid H.264 stream bytes into derived MP4 clip (clip_ch01_001.mp4).
- Display Artifact Provenance: Source Image EVD-2026-001 | Offset: 0x01A400 | Length: 14.2 MB | SHA-256: 4f8b...

[Demo Step 5: Timeline Normalization]
- Extract raw in-stream timestamp: 2026-08-25 14:22:05 (Raw Channel Clock).
- Apply clock drift correction (-00:02:14) -> UTC Normalized: 2026-08-25 14:19:51 UTC.
- Populate multi-camera master timeline grid.

[Demo Step 6: Secondary AI Analytical Triage & Human Review]
- ONNX Runtime executes YOLOv8 model on frame thumbnails at 1 fps.
- AI detects "vehicle" candidate at timestamp 14:20:02 with confidence 0.88.
- Display label: "AI-Generated Analytical Finding (Probabilistic)".
- Investigator clicks "Verify" -> Status updated to VERIFIED.

[Demo Step 7: Forensic Evidence Report Generation]
- Export structured HTML/PDF Forensic Evidence Report.
- Report features Evidence Hashes, Processing Log, Derived Artifact Hashes, Offset Provenance Table, Human-Reviewed AI Summary, and Limitation Disclaimers.
```

---

## 3. Ground-Truth Validation Metrics Framework

Locus measures system accuracy using controlled ground-truth test datasets. Metrics are strictly categorized:

| Benchmark Category | Target Standard | Measured Value | Validation Dataset |
| :--- | :--- | :--- | :--- |
| **Vendor Detection Accuracy** | 100% on validated OEM profiles | *To be measured* | 10 test images (Dahua, Hikvision) |
| **Intact Index Extraction** | 100% of non-deleted index clips | *To be measured* | Synthetic + real CCTV drive dumps |
| **Surviving Deleted Carving** | ≥ 80% recovery of un-overwritten sectors | *To be measured* | Controlled deleted sector images |
| **Overwritten Sector Handling**| 100% explicit UNRECOVERABLE flag | *To be measured* | Intentionally overwritten sector blocks |
| **Timestamp Preservation** | 100% parity with raw stream header | *To be measured* | Multi-camera test feeds with known drift |
| **AI Precision / Recall** | Precision ≥ 85% on clear frames | *To be measured* | Standard CCTV resolution test videos |

> *Note:* All "To be measured" metrics require empirical testing against laboratory ground-truth datasets before publication.
