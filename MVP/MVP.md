# LOCUS — MVP Architecture & Features Specification

**Back to [[locus]]**

---

## 1. MVP Overview

The **Locus MVP** is structured into **8 core backend feature specifications** (+ 1 forensic evidence reporting module), mapping directly to the official PS 26150 requirements. Each module is designed around the central principle: **Locus prioritizes forensic correctness over successful-looking output.**

---

## 2. End-to-End Forensic Data Pipeline

```
[1. Forensic Image Ingestion] ──► [2. Device Layout Identification] ──► [3. Storage & Header Analysis]
                                                                                  │
                                                                                  ▼
[6. Secondary AI Triage] ◄─────── [5. Timeline Normalization] ◄─── [4. Stream Carving & Remuxing]
           │
           ▼
[7. Search & Filter Querying] ──► [8. Cryptographic Hash Parity] ──► [9. Forensic Evidence Report]
```

---

### Key Workflow Responsibilities

#### 1. Pre-Acquired Image Ingestion & Baseline Hashing
- **What happens:** The investigator loads a pre-acquired forensic disk image (`.dd`, `.raw`, `.img`). Locus opens file handles in strict **Read-Only mode** (`rb` mode) and streams 64 KB chunks to compute baseline `SHA-256` and `MD5` cryptographic hashes.
- *Note:* Software read-only handling prevents application-level write operations; physical hardware write-blocking remains a separate upstream acquisition process.

#### 2. Device & File System Layout Identification
- **What happens:** Locus scans sector header boundaries looking for candidate signature bytes (e.g., `DHAV` for Dahua profiles, `HKFS` for Hikvision profiles). It estimates candidate vendor identification, partition layout, and confidence level (`KNOWN`, `LIKELY`, `UNKNOWN`, `UNSUPPORTED`, `AMBIGUOUS`). (Note: Dedicated CP Plus adapter validation is scheduled for Phase 2 — not in MVP).

#### 3. Storage & Header Analysis
- **What happens:** Locus unpacks vendor-specific binary sector headers (such as validated Dahua DHAV frame wrappers or Hikvision block headers) using verified format definitions. It constructs a **Master Sector Map** in SQLite mapping frame boundaries and channel IDs.

#### 4. Video Stream Carving & Remuxing
- **What happens:** The background Python worker reads raw disk sectors according to the Master Sector Map. If filesystem indexes are missing, it executes metadata-guided sector carving. Valid H.264/H.265 frames are remuxed into web-playable `.mp4` clips via PyAV/FFmpeg using zero-transcoding stream copying. Stream recovery status is explicitly recorded (`RECOVERED`, `PARTIAL`, `FRAGMENTED`, `CORRUPTED`, `UNRECOVERABLE`).

#### 5. Multi-Camera Master Timeline Synchronization
- **What happens:** Locus preserves original in-stream timestamps while calculating timezone interpretations and clock drift offsets. A normalized master UTC timeline synchronizes multi-camera grid playback without mutating source evidence.

#### 6. Local Secondary AI Video Analytics (ONNX Engine)
- **What happens:** As video clips are validated, Locus samples frames at configurable intervals (e.g., 1 fps) and runs lightweight YOLOv8 models locally via ONNX Runtime to detect object (`person`, `vehicle`) and motion candidate tags. Detection bounding boxes and confidence scores are indexed in SQLite with mandatory Human-in-the-Loop review flags (`Verified`, `Rejected`, `Unreviewed`).

#### 7. Evidence Search & Event Filtering
- **What happens:** The investigator executes queries (channel + timestamp range + object class + verification status). Locus retrieves indexed SQLite records to populate an interactive thumbnail gallery and timeline overlays.

#### 8. Cryptographic Hash Verification & Integrity Export
- **What happens:** Upon exporting derived clips, Locus re-verifies source image baseline hashes, calculates output artifact `SHA-256`/`MD5` hashes, and generates a cryptographic provenance record (`.sync.json`) detailing source image ID, byte offset, sector length, and parser version.

#### 9. Forensic Evidence Report Export
- **What happens:** Compiles case metadata, baseline hashes, carved media hash parity tables, full artifact provenance traces, human-reviewed AI findings, and uncertainty disclaimers into a structured HTML/PDF Forensic Evidence Report.

---

## 3. Feature Modules Index

Each feature document inside `MVP/features/` defines low-level technical specifications:

| # | Feature Folder | Document Link | Core Technical Focus | Status |
| :--- | :--- | :--- | :--- | :--- |
| 01 | `disk-imaging` | [disk-imaging.md](./features/disk-imaging/disk-imaging.md) | Pre-acquired forensic image ingestion (`.dd`, `.raw`), software read-only handles, dual SHA-256/MD5 baseline hashing | **Updated** |
| 02 | `device-identification` | [device-identification.md](./features/device-identification/device-identification.md) | Signature scanner, vendor confidence scoring (`KNOWN`, `LIKELY`, `UNKNOWN`, `UNSUPPORTED`, `AMBIGUOUS`), header validation | **Updated** |
| 03 | `filesystem-parsing` | [filesystem-parsing.md](./features/filesystem-parsing/filesystem-parsing.md) | MBR/GPT partition analysis, Dahua/Hikvision header parsing, Master Sector Map schema | **Updated** |
| 04 | `video-carving` | [video-carving.md](./features/video-carving/video-carving.md) | Index parsing, sector carving fallback, GOP alignment, zero-transcoding remuxing, status tracking | **Updated** |
| 05 | `timeline-sync` | [timeline-sync.md](./features/timeline-sync/timeline-sync.md) | Raw timestamp preservation, timezone interpretation, clock drift calibration, UTC master timeline | **Updated** |
| 06 | `ai-analytics` | [ai-analytics.md](./features/ai-analytics/ai-analytics.md) | Local ONNX Runtime (YOLOv8) secondary triage, object/face/motion candidate indexing, Human-in-the-Loop review | **Updated** |
| 07 | `evidence-search` | [evidence-search.md](./features/evidence-search/evidence-search.md) | Parameterized SQLite search queries across timestamps, camera channels, and verified AI tags | **Updated** |
| 08 | `evidence-hashing` | [evidence-hashing.md](./features/evidence-hashing/evidence-hashing.md) | Export hash parity verification, artifact SHA-256 calculation, cryptographic provenance sidecar export | **Updated** |
| 09 | `forensic-reporting` | [forensic-reporting.md](./features/forensic-reporting/forensic-reporting.md) | HTML/PDF Forensic Evidence Report generation (WeasyPrint + Jinja2); hash parity table, artifact provenance, human-reviewed AI findings, uncertainty disclaimers | **New** |

---

## 4. Layered System Architecture Diagram

```
┌───────────────────────────────────────────────────────────────────────────┐
│                          LOCUS ELECTRON DESKTOP SHELL                     │
│  ┌─────────────────────────────────────────────────────────────────────┐  │
│  │                    REACT + TYPESCRIPT FRONTEND                      │  │
│  │   (Case Intake, Sector Hex Viewer, Sync Timeline, Report Generator) │  │
│  └──────────────────────────────────┬──────────────────────────────────┘  │
└─────────────────────────────────────┼─────────────────────────────────────┘
                                      │ HTTP REST / WebSockets (127.0.0.1:8000)
                                      ▼
┌───────────────────────────────────────────────────────────────────────────┐
│                      FASTAPI PYTHON BACKEND ENGINE                        │
│                                                                           │
│  ┌────────────────────────┐  ┌────────────────────────┐  ┌─────────────┐  │
│  │ 01. Evidence Ingestion │  │ 02. Device Layout ID   │  │ 03. FS Map  │  │
│  └────────────────────────┘  └────────────────────────┘  └─────────────┘  │
│  ┌────────────────────────┐  ┌────────────────────────┐  ┌─────────────┐  │
│  │ 04. Stream Carver      │  │ 05. Timeline Sync Bus  │  │ 06. ONNX AI │  │
│  └────────────────────────┘  └────────────────────────┘  └─────────────┘  │
│  ┌────────────────────────┐  ┌────────────────────────┐  ┌─────────────┐  │
│  │ 07. Evidence Search    │  │ 08. Hash Parity Engine │  │ 09. Report  │  │
│  └────────────────────────┘  └────────────────────────┘  └─────────────┘  │
└─────────────────────────────────────┬─────────────────────────────────────┘
                                      │
                                      ▼
                           ┌─────────────────────┐
                           │ SQLite Database     │
                           │ (`case_meta.db`)    │
                           └─────────────────────┘
```
