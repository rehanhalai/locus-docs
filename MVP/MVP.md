# LOCUS — MVP Architecture & Features Specification

**Back to [[locus]]**

---

## MVP Overview

The **Locus MVP** is structured into **8 core backend feature specifications** (+ 1 deferred reporting module), mapping directly to the official PS 26150 requirements. Each feature contains the explanation and technical architecture of the MVP.

---

### How the features are structured to collaborate with each other

```text
[1. Physical Acquisition & Ingestion] ──► [2. Device Identification] ──► [3. Sector Header Parsing]
                                                                                   │
                                                                                   ▼
[6. AI Analytics & Indexing] ◄──────── [5. Timeline Synchronization] ◄── [4. Sector Video Carving]
           │
           ▼
[7. Evidence Search & Querying] ──────► [8. Hash Verification & Export] ──► [9. Forensic PDF Report*]
```

---

#### 1. Physical Acquisition & Case Ingestion
- **What happens:** The investigator either acquires a live raw hard drive via hardware write-blocker using the embedded `dc3dd` engine (requires Administrator/Root privileges) or loads a pre-existing forensic image (`.dd`/`.raw`). Locus locks the image in **strict Read-Only mode** and generates baseline `SHA-256` and `MD5` cryptographic hashes.

#### 2. Device & File System Identification
- **What happens:** Locus scans the raw sector headers looking for manufacturer "magic signatures" (e.g., `DHAV` for Dahua/CP Plus, `HKFS` for Hikvision). It rapidly detects the DVR model, partition layout, sector size, and camera channel count.

#### 3. Sector Header Parsing & Mapping
- **What happens:** Locus scans disk sectors, unpacks proprietary 32-byte binary headers using Python `struct.unpack()`, and builds a **Master Sector Map** in SQLite (`stream_headers` table) that maps out where every camera channel's frames begin and end.

#### 4. Sector Video Carving & Stream Remuxing
- **What happens:** The background Python worker reads raw sectors according to the Master Sector Map, strips proprietary wrapper headers, snaps to the nearest I-Frame (GOP alignment), and remuxes raw H.264/H.265 frames into web-playable `.mp4` clips via PyAV/FFmpeg using **zero-transcoding stream copy**.

#### 5. Multi-Camera Master Timeline Sync
- **What happens:** Locus normalizes timestamp drifts across different camera channels using non-destructive virtual calibration layers (`timeline_calibrations`). A 60 Hz master clock coordinates multi-camera grid playback without modifying source evidence.

#### 6. Local AI Video Analytics (ONNX Engine)
- **What happens:** As clips are carved, Locus uses OpenCV MOG2 to detect motion voids (skipping dead time), then runs lightweight YOLOv8 models locally via ONNX Runtime to detect persons and vehicles, indexing event timestamps into SQLite.

#### 7. Evidence Search & Event Filtering
- **What happens:** The investigator executes fast parameterized queries (camera + time range + object type + confidence). Locus queries the indexed SQLite database with sub-second latency and populates a clickable thumbnail gallery and timeline heatmap markers.

#### 8. Cryptographic Hash Verification & Integrity Export
- **What happens:** Upon export, Locus slices the carved video using zero-transcoding copy, calculates the output SHA-256/MD5 artifact hashes, and generates a cryptographic audit sidecar (`.sync.json`) linking the clip directly back to the baseline source image hash. On-demand whole-disk verification (`POST /api/verify`) is available for case closure.

#### 9. Forensic Evidence PDF Report Export *(Future / UI Reporting Module)*
- **What happens:** Compiles case metadata, baseline disk hashes, carved clip tables, AI detection summaries, and investigator audit logs into a legally formatted PDF report.
- **Specification Status:** *Note: This feature is deferred from the current low-level backend technical specifications due to its straightforward UI/template nature (HTML/PDF rendering), but remains an integral component of the product roadmap.*

---

## Feature Modules Index

Each feature below has its own dedicated folder inside `MVP/features/` with a comprehensive specification document. Reading these documents in order gives a complete understanding of the entire Locus backend architecture.

| # | Feature Folder | Document | Summary | Status |
| :--- | :--- | :--- | :--- | :--- |
| 01 | [[disk-imaging]] | [disk-imaging.md](./features/disk-imaging/disk-imaging.md) | Live physical drive acquisition via embedded `dc3dd`, bad sector defense, and dual SHA-256/MD5 baseline hashing | **Complete** |
| 02 | [[device-identification]] | [device-identification.md](./features/device-identification/device-identification.md) | Automatic OEM & magic byte signature detection (Dahua, Hikvision, CP Plus, etc.) | **Complete** |
| 03 | [[filesystem-parsing]] | [filesystem-parsing.md](./features/filesystem-parsing/filesystem-parsing.md) | Binary header decoding, sector mapping, and building the Master Sector Map in SQLite | **Complete** |
| 04 | [[video-carving]] | [video-carving.md](./features/video-carving/video-carving.md) | Sector-level carving of raw H.264/H.265 frames with GOP alignment and zero-transcoding remuxing | **Complete** |
| 05 | [[timeline-sync]] | [timeline-sync.md](./features/timeline-sync/timeline-sync.md) | Multi-camera timestamp normalization, non-destructive calibration layer, and 60 Hz master clock | **Complete** |
| 06 | [[ai-analytics]] | [ai-analytics.md](./features/ai-analytics/ai-analytics.md) | Local YOLOv8 object detection (persons, vehicles) & motion void indexing via ONNX Runtime | **Complete** |
| 07 | [[evidence-search]] | [evidence-search.md](./features/evidence-search/evidence-search.md) | SQLite query layer for filtering, searching, and browsing AI detection events | **Complete** |
| 08 | [[evidence-hashing]] | [evidence-hashing.md](./features/evidence-hashing/evidence-hashing.md) | Cryptographic hash verification, zero-transcoding export, and audit trail sidecar generation | **Complete** |
| 09 | `forensic-reporting` | *Deferred* | Automated Forensic Evidence PDF report export with hash parity tables | *Future / Low Complexity* |

---

## System Overview Diagram

```
┌───────────────────────────────────────────────────────────────────────────┐
│                          LOCUS ELECTRON DESKTOP SHELL                     │
│  ┌─────────────────────────────────────────────────────────────────────┐  │
│  │                     REACT + SHADCN UI FRONTEND                      │  │
│  │   (Case Intake, Feature Tabs, Video Grid, Timeline, PDF Export)     │  │
│  └──────────────────────────────────┬──────────────────────────────────┘  │
└─────────────────────────────────────┼─────────────────────────────────────┘
                                      │ HTTP REST / WebSockets (localhost:8000)
                                      ▼
┌───────────────────────────────────────────────────────────────────────────┐
│                      FASTAPI PYTHON BACKEND ENGINE                        │
│                                                                           │
│  ┌────────────────────────┐  ┌────────────────────────┐  ┌─────────────┐  │
│  │ 01. Acquisition (dc3dd)│  │ 02. Device ID & Scanners│  │ 03. FS Parser│  │
│  └────────────────────────┘  └────────────────────────┘  └─────────────┘  │
│  ┌────────────────────────┐  ┌────────────────────────┐  ┌─────────────┐  │
│  │ 04. Sector Carver      │  │ 05. Timeline Sync Bus  │  │ 06. ONNX AI │  │
│  └────────────────────────┘  └────────────────────────┘  └─────────────┘  │
│  ┌────────────────────────┐  ┌────────────────────────┐                   │
│  │ 07. Evidence Search    │  │ 08. Hash Verification  │                   │
│  └────────────────────────┘  └────────────────────────┘                   │
└─────────────────────────────────────┬─────────────────────────────────────┘
                                      │
                                      ▼
                           ┌─────────────────────┐
                           │ SQLite Database     │
                           │ (`forensics.db`)    │
                           └─────────────────────┘
```
