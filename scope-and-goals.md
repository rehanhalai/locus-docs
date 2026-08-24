# Locus Project Scope & Goals

**Back to [[locus]]**

---

## Project Objective

To design and build **Locus**, a standalone, cross-platform desktop digital forensics application (Electron + React + FastAPI + SQLite + ONNX) that automates evidence acquisition, sector-level video carving, multi-camera timeline synchronization, cryptographic evidence validation, local AI analytics, and legal report generation for surveillance evidence.

---

## 🎯 Tier 1: MUST-HAVE (Core MVP Scope for Sept 10–12 Submission)

These 5 core deliverables are non-negotiable for the SIH submission:

### 1. Standalone Desktop Shell & Case Intake
- Electron desktop application shell launching an embedded Python FastAPI background server.
- Native desktop file picker (`dialog.showOpenDialog`) to select raw disk dumps (`.dd` / `.raw`).
- Automatic `SHA-256` and `MD5` baseline cryptographic hash calculation upon ingestion.
- Green `Read-Only Write-Block Safeguard` security indicator on UI.

### 2. Sector Carving Engine (Dahua & Hikvision)
- Read-only binary file parser (`open(filepath, 'rb')`).
- Sector-level pattern scanner searching for Dahua (`DHAV`) and Hikvision magic headers in raw disk images.
- PyAV / FFmpeg remuxing engine to convert raw carved H.264/H.265 streams into standard `.mp4` video clips.

### 3. Evidence Registry & Video Player UI
- React + ShadcnUI dashboard displaying carved video clips with individual SHA-256 extraction hashes.
- HTML5 Video Player featuring play/pause, speed controls, frame-by-frame stepping, and timestamp overlays.

### 4. Local AI Analytics (ONNX Runtime)
- Frame sampler extracting 1 frame per second from carved video clips.
- Local ONNX Runtime inference using `yolov8n.onnx` (~12 MB model) running on CPU.
- Object detection indexing (`person`, `vehicle`) saved to local SQLite `timeline_events` table.
- Timeline search filter bar (filter by camera channel or object class).

### 5. Court-Ready PDF Evidence Report
- FastAPI endpoint generating audit-compliant PDF reports.
- Includes Case ID, Baseline Image Hash, Carved Clips Hash Parity Table, AI Detection Summary, and Investigator Signature block.

---

## 🚀 Tier 2: STRETCH GOALS (Nice-to-Have if Time Permits)

- **CP Plus OEM Adapter:** Adding sector header scanner for CP Plus proprietary formats.
- **Multi-Camera Synchronized Player:** Scrubbing 2 to 4 camera feeds simultaneously on a single master timeline bar.
- **Facial Detection Matching:** Local FaceNet embedding comparison matching a suspect photo against carved video frame thumbnails.

---

## 🛑 Tier 3: EXPLICIT OUT-OF-SCOPE (Non-Goals)

To prevent scope creep and ensure on-time delivery by September 20:

- **No Live Network IP Camera Streaming:** Locus is strictly an *offline disk forensics tool*, not a live VMS camera streaming server.
- **No Cloud Backend / Cloud Sync:** Purely local application using local SQLite (`forensics.db`); zero external cloud APIs or servers.
- **No Physical Hardware Drive Repair:** Assumes disk images are already physically imaged (`.dd` file); does not perform physical mechanical repair on damaged hard drive heads.
- **No Heavy LLM Chatbots:** Out of scope for initial MVP submission.

---

## 📊 Acceptance Criteria & Definition of Done

1. **Ingestion & Hash Parity:** App successfully ingests a raw `.dd` file and calculates SHA-256 hash without modifying the source file.
2. **Carving Success:** App recovers deleted video fragments from sample `.dd` files and converts them to playable `.mp4` clips.
3. **Local AI Execution:** YOLOv8 runs locally via ONNX Runtime on CPU and logs `person` / `vehicle` detections into SQLite.
4. **Report Export:** App exports a court-ready PDF evidence report in less than 5 seconds.
5. **Cross-Platform Launch:** Desktop app launches cleanly on Windows 10/11 (`.exe`) and Ubuntu Linux (`.AppImage`).
