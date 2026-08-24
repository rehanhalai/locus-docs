# LOCUS — MVP Architecture & Features Specification

**Back to [[locus]]**

---

## MVP Overview

The **Locus MVP** is structured into **8 modular feature specifications**, mapping directly to the official PS 26150 requirements. Each feature contains the explaination and how the featue is structured in the MVP.

---

### How the features will be structured to collaborate with each others

[1. Physical Acquisition] ──► [2. Case Ingestion & Hashing] ──► [3. Device Identification]

 [6. Multi-Camera Timeline] ◄── [5. Local AI Analytics] ◄── [4. Sector Video Carving]

 [7. Evidence Search] ────► [8. Cryptographic Audit] ──────► [9. Court PDF Report]

---

#### 1. Physical Acquisition _(Pre-App Hardware Step)_

- **What happens:** The investigator connects the seized evidence hard drive to a physical hardware write-blocker and creates a 1-to-1 byte-level bit-stream disk image file (`case_101.dd` or `.raw`).

#### 2. Case Intake & Baseline Hashing

- **What happens:** The investigator opens **Locus**, clicks **"New Case"**, and selects the `.dd` file using the native file picker. Locus locks the file in **strict Read-Only mode** and instantly calculates baseline `SHA-256` and `MD5` cryptographic hashes to freeze the evidence state.

#### 3. Device & File System Identification

- **What happens:** Locus scans the raw sector headers of the `.dd` file looking for manufacturer "magic signatures" (e.g., `DHAV` for Dahua, `HKFS` for Hikvision). Within 1 second, it detects the DVR model, partition structure, and camera channel count.

#### 4. Sector Video Carving & Stream Remuxing

- **What happens:** The background Python worker scans raw drive sectors (including deleted/unallocated space), finds orphaned video frame headers, extracts raw H.264/H.265 chunks, and remuxes them using PyAV/FFmpeg into standard web-playable `.mp4` clips.

#### 5. Local AI Video Analytics (ONNX Engine)

- **What happens:** As clips are carved, our local ONNX Runtime engine extracts 1 frame per second and runs YOLOv8 to detect targets (_people, vehicles, objects, motion_). It indexes these tags and timestamps into the local SQLite database.

#### 6. Multi-Camera Master Timeline Sync

- **What happens:** Locus normalizes timestamp drifts across different camera channels and plots them on a single master timeline bar. The investigator can scrub the timeline, playing multiple camera feeds synchronously at the exact same real-world second.

#### 7. Evidence Search & Event Filtering

- **What happens:** The investigator types a filter query in the React search bar (e.g., _"Show all red vehicles on Camera 2 between 2:00 PM and 3:00 PM"_). Locus queries SQLite in 2 milliseconds and highlights match markers directly on the timeline player.

#### 8. Cryptographic Hash Verification & Chain-of-Custody Audit

- **What happens:** The investigator clicks **"Verify Integrity"**. Locus recalculates hashes and verifies 100% match parity against the original baseline ingestion hashes, proving the evidence was never tampered with.

#### 9. Court-Ready Forensic PDF Report Export

- **What happens:** With 1 click, Locus compiles case metadata, baseline disk hashes, carved clip tables, AI detection summaries, and investigator audit logs into a legally formatted, court-ready PDF report ready for submission to judges.

## Feature Modules Index

- **[[device-identification]]** — Automatic OEM & hardware signature detection (Dahua, Hikvision, CP Plus, etc.)
- **[[disk-imaging]]** — Bit-stream raw disk dump ingestion (`.dd`/`.raw`) with write-block safeguard
- **[[filesystem-parsing]]** — Unmasking proprietary DVR filesystems (DHFS, Hikvision raw layouts) & container headers
- **[[video-carving]]** — Sector-level carving of deleted or damaged H.264/H.265 video fragments
- **[[timeline-sync]]** — Multi-camera timestamp normalization & master synchronized timeline player
- **[[evidence-hashing]]** — Dual SHA-256 / MD5 cryptographic evidence verification & immutable audit trail
- **[[ai-analytics]]** — Local YOLOv8 object detection (people, vehicles) & motion indexing via ONNX Runtime
- **[[forensic-reporting]]** — Automated court-ready PDF forensic report export with hash parity tables

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
│  ┌──────────────────┐  ┌──────────────────┐  ┌─────────────────────────┐  │
│  │ Device Identification│ 02. Acquisition │  │ 03. FS & Header Parser  │  │
│  └──────────────────┘  └──────────────────┘  └─────────────────────────┘  │
│  ┌──────────────────┐  ┌──────────────────┐  ┌─────────────────────────┐  │
│  │ 04. Sector Carver│  │ 05. Timeline Sync│  │ 06. Cryptographic Hash  │  │
│  └──────────────────┘  └──────────────────┘  └─────────────────────────┘  │
│  ┌──────────────────┐  ┌──────────────────┐                               │
│  │ 07. ONNX AI Engine│ │ 08. PDF Exporter │                               │
│  └──────────────────┘  └──────────────────┘                               │
└─────────────────────────────────────┬─────────────────────────────────────┘
                                      │
                                      ▼
                           ┌─────────────────────┐
                           │ SQLite Database     │
                           │ (`forensics.db`)    │
                           └─────────────────────┘
```
