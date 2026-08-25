# Locus Project Scope, MVP Boundaries & Development Roadmap

**Back to [[locus]]**

---

## 1. Project Objective

To architect and build **Locus**, a specialized desktop surveillance forensic analysis platform for pre-acquired disk images (`.dd`, `.raw`, `.img`). Locus automates storage layout detection, vendor-aware stream recovery, media validation, timestamp normalization, artifact provenance tracking, secondary AI video triage, and structured forensic reporting.

---

## 2. 🎯 Tier 1: Core MVP Scope (Immediate Deliverable)

The MVP scope focuses on pre-acquired forensic image ingestion and deterministic read-only analysis:

### 1. Pre-Acquired Image Ingestion & Dual Cryptographic Hashing
- Ingestion of raw disk dumps (`.dd`, `.raw`, `.img`) via desktop file picker.
- Automated streaming calculation of baseline `SHA-256` and `MD5` hashes upon evidence registration.
- Software read-only guard enforcement (`rb` mode file handles).
- *Note:* Software read-only handling prevents application-level writes, but is explicitly distinguished from physical hardware write-blocking.

### 2. Device Layout Identification & Deeply Validated Adapters
- Sector-aligned magic header scanner across disk image boundaries.
- **Dahua Adapter (`DahuaDHAVAdapter`):** Index-aware stream recovery + sector carving fallback for Dahua DHAV layouts.
- **Hikvision Adapter (`HikvisionHKFSAdapter`):** HKFS partition index parsing + sector stream extraction for Hikvision HKFS layouts.
- Explicit `UNKNOWN` classification for unrecognized signatures; no forced adapter matching.

### 3. Media Integrity & Recovery Validation Engine
- Demux and stream continuity check via PyAV/FFmpeg bindings.
- Assign recovery statuses: `RECOVERED`, `PARTIAL`, `FRAGMENTED`, `CORRUPTED`, `UNRECOVERABLE`, `UNSUPPORTED`.
- Conversion of valid stream payloads to derived `.mp4` media clips with individual artifact SHA-256 hashes.

### 4. Raw & UTC Normalized Timeline Engine
- Extraction and preservation of raw in-stream metadata timestamps.
- Timezone offset interpretation and clock drift adjustment to construct a normalized master UTC timeline.
- Interactive multi-camera synchronized player UI in React.

### 5. Secondary AI Triage Layer (ONNX Runtime)
- Configurable frame sampling (e.g., 1 fps) on validated video clips.
- Local CPU inference using ONNX Runtime (`yolov8n.onnx`) for object (`person`, `vehicle`) and motion candidate detection.
- Storage of detection bounding boxes, confidence scores, and frame timestamps in SQLite (`case_meta.db`).
- Mandatory Human-in-the-Loop review status (`Verified`, `Rejected`, `Unreviewed`).

### 6. Forensic Evidence Report Generation
- Generation of structured HTML and PDF Forensic Evidence Reports.
- Includes Case Details, Baseline Evidence Hashes, Carved/Extracted Media Hash Parity Table, Full Artifact Provenance (Offset, Length, Adapter Version), AI Analytical Findings Summary, and Investigator Review Log.

---

## 3. 🚀 Tier 2: Phase 2 Roadmap (Medium-Term Development)

- **Additional Vendor Adapters:** Deep laboratory research and validation for CP Plus, Honeywell, TP-Link, Godrej, Uniview, and Matrix proprietary layouts.
- **Advanced Fragment Reconstruction:** Multi-stream fragment reordering based on GOP continuity, timestamp monotonicity, and frame header signatures.
- **Enhanced AI Analytics:** Facial detection bounding box clustering and vehicle color classification models.

---

## 4. 🔮 Tier 3: Phase 3 Roadmap (Long-Term Development)

- **Physical Hardware Acquisition:** Upstream physical drive acquisition workflow and direct hardware write-blocker integration.
- **Proprietary Evidence Containers:** Native ingestion of `.E01` (Expert Witness Format) and `.AFF4` forensic container files.
- **Advanced Filesystem Research:** Reconstruction of severely damaged or zero-byte filesystem metadata tables.
- **Face Recognition (identity-similarity embedding):** Requires dedicated legal/privacy review before implementation given Indian data protection and privacy jurisprudence (Puttaswamy). Out of scope for MVP and Phase 2.

---

## 5. 🛑 What Locus Does NOT Do (Explicit Non-Goals)

To maintain technical defensibility and prevent misrepresentation, Locus explicitly declares:

1. **No Hardware Write-Blocking in Software:** Locus performs application-level read-only analysis. It does **not** replace physical hardware write-blockers for drive acquisition.
2. **No Recovery of Overwritten Data:** Locus cannot recover or reconstruct data from drive sectors that have been physically overwritten by new video recordings.
3. **No Unvalidated Format Inventing:** Locus does not invent parser support for vendor models without verified laboratory test data.
4. **No Automated Legal Admissibility Guarantees:** Locus provides evidence reports and cryptographic proof; it does not issue legal guarantees or claims of court admissibility.
5. **No Cloud Offloading:** Locus processes evidence 100% locally; it does not transmit evidence or video streams to cloud servers.
6. **No Automated Suspect Identification:** AI object and face detection results are secondary triage suggestions requiring investigator verification, not automated proof of identity.
7. **No Live Network Camera Capture:** Locus is an offline post-event disk image analysis tool, not a live network Video Management System (VMS).
8. **No Physical Hard Drive Repair:** Locus does not repair damaged mechanical disk heads, firmware chips, or bad sectors.

---

## 6. Acceptance Criteria & Definition of Done

1. **Evidence Integrity:** Source disk image SHA-256 and MD5 hashes computed at registration match identical calculations executed post-analysis.
2. **Deterministic Layout Detection:** Correctly identifies Dahua DHAV or Hikvision HKFS headers on ground-truth test datasets, or explicitly flags as `UNKNOWN`.
3. **Stream Recovery & Validation:** Extracts intact recordings and surviving fragments, assigning explicit status (`RECOVERED`, `PARTIAL`, `CORRUPTED`, `UNRECOVERABLE`).
4. **Full Provenance Tracking:** Every derived clip in the workspace is traceable to exact source image ID, byte offset, sector length, and parser version.
5. **Reproducibility:** Re-running analysis on identical evidence images produces identical artifact hashes and timeline structures.
6. **Report Accuracy:** Generated Forensic Evidence Reports contain zero hallucinated timestamps or unverified claims.
