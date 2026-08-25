# Locus Project Scope & Goals

**Back to [[locus]]**

---

## Project Objective

To design and build **Locus**, a standalone, cross-platform desktop digital forensics application (Electron + React + FastAPI + SQLite + ONNX) that automates evidence acquisition, sector-level video carving, multi-camera timeline synchronization, cryptographic evidence validation, local AI analytics, and legal report generation for surveillance evidence.

---

## 🎯 Tier 1: MUST-HAVE (Core MVP Scope for Sept 10–12 Submission)

These core deliverables correspond to the 8 feature specifications and are non-negotiable for the SIH submission:

### 1. Standalone Desktop Shell & Case Intake (Features 01 & 02)
- Electron desktop application shell launching an embedded Python FastAPI background server.
- **Live Physical Drive Acquisition:** Embedded `dc3dd` engine to clone raw hard drives (Requires hardware write-blocker and Admin/Root execution). *(Note: For the MVP phase, investigators must have sufficient local storage to hold the full `.dd` clone.)*
- **File Ingestion:** Native desktop file picker to select existing raw disk dumps (`.dd` / `.raw`).
- Automatic `SHA-256` and `MD5` baseline cryptographic hash calculation upon ingestion.
- **2-Phase Identification:** Automatic OEM & file system identification via magic headers followed by structural layout validation.

### 2. Sector Carving Engine (Features 03 & 04)
- Proprietary 32-byte binary header decoding to build a Master Sector Map in SQLite.
- Sector-level carving to recover damaged/deleted H.264/H.265 frames.
- **SPS/PPS Header Injection:** Dynamically injects Sequence Parameter Set (SPS) and Picture Parameter Set (PPS) headers into raw NAL units.
- PyAV / FFmpeg remuxing engine to convert raw carved video into standard web-playable `.mp4` clips using zero-transcoding.

### 3. Multi-Camera Timeline Sync (Feature 05)
- Non-destructive timestamp calibration layer.
- 60 Hz master clock synchronizing playback of up to 4 camera feeds concurrently on a single master timeline using **Ref-Based Direct DOM Manipulation (`requestAnimationFrame`)** to prevent React DOM thrashing.

### 4. Local AI Analytics & Evidence Search (Features 06 & 07)
- Local ONNX Runtime YOLOv8 inference running on CPU to detect `person` and `vehicle`.
- OpenCV MOG2 motion void detection.
- Fast SQLite querying allowing investigators to filter events by object, time, or channel in milliseconds.

### 5. Integrity Audit & Forensic Evidence Report (Feature 08 & 09)
- Cryptographic hash verification upon export (`.sync.json` audit sidecar).
- FastAPI endpoint generating audit-compliant PDF reports with Hash Parity Tables and AI summaries.

---

## 🚀 Tier 2: STRETCH GOALS (Nice-to-Have if Time Permits)

- **CP Plus OEM Adapter:** Adding sector header scanner for CP Plus proprietary formats (if different from Dahua).
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
4. **Report Export:** App exports a Forensic Evidence PDF report in less than 5 seconds.
5. **Cross-Platform Launch:** Desktop app launches cleanly on Windows 10/11 (`.exe`) and Ubuntu Linux (`.AppImage`).
