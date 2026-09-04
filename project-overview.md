# Project Overview: LOCUS

**Project Name:** Locus  
**Tagline:** Unified Multi-Vendor DVR/NVR Forensic Analysis Suite  
**SIH Problem Statement ID:** 26150  
**Target Organization:** National Technical Research Organisation (NTRO)  
**Theme:** Cybersecurity & Digital Forensics | **Category:** Software  

---

## Executive Summary

**Locus** is a next-generation, cross-platform desktop forensic application designed to solve the critical challenge of surveillance evidence fragmentation in law enforcement and intelligence operations. 

Modern CCTV and DVR/NVR systems from major manufacturers—such as **Dahua, CP Plus, Honeywell, HIKVISION, TP-Link, Godrej, Uniview, and Matrix**—use proprietary file systems, raw disk structures, non-standard container headers, and custom video stream encodings. During criminal investigations, forensic officers are forced to juggle multiple vendor-specific playback utilities, struggle with timestamp drift across cameras, face extreme difficulty in recovering deleted footage, and risk evidence contamination.

**Locus** bridges this gap by offering a unified, vendor-agnostic software suite. It automates raw sector parsing, deleted video carving, multi-camera timeline synchronization, cryptographic evidence verification (`SHA-256` & `MD5`), and computer vision AI analytics—delivering forensic evidence reports from a single desktop interface.

---

## Key Features & Core Capabilities

1. **Physical Acquisition & Case Ingestion**
   - Captures physical raw DVR disks via embedded `dc3dd` engine (requires physical hardware write-blocker and Administrator/Root privileges) or loads existing `.dd` forensic image files.

2. **Automated OEM & File System Identification**
   - **2-Phase Validation:** Initially detects proprietary magic header signatures (`DHAV`, `HKFS`), followed by structural layout validation. Unrecognized structures are strictly flagged as `UNKNOWN`.

3. **Sector Header Parsing & Mapping**
   - Decodes proprietary 32-byte binary headers to construct a Master Sector Map of all interleaved camera channels.

4. **Sector-Level Video Carving & Deleted Recovery**
   - Bypasses corrupted or missing file systems. Scans raw drive sectors to carve deleted, fragmented, or damaged H.264/H.265 video using PyAV/FFmpeg zero-transcoding remuxing.

5. **Multi-Camera Synchronized Master Timeline**
   - Calibrates internal DVR clock drifts non-destructively. Original raw DVR timestamps are strictly preserved, while synchronizing heterogeneous camera channels into a unified chronological master timeline driven by a 60 Hz internal clock.

6. **Local AI Video Analytics (ONNX Engine)**
   - **Motion-Gated Pipeline:** Uses OpenCV MOG2 as a high-speed pre-filter to detect motion, ensuring the YOLOv8 model via ONNX Runtime only processes frames with actual activity. AI outputs include confidence scores and act as an investigative aid requiring human verification.

7. **Evidence Search & Event Filtering**
   - High-speed SQLite filtering allowing investigators to search specific AI detections across specific cameras in milliseconds.

8. **Cryptographic Integrity & Audit Logging**
   - Automatically calculates and verifies `SHA-256` and `MD5` hashes at disk ingestion, file carving, extraction, and report export—providing strict integrity, provenance, and processing history.

9. **Forensic Evidence Reporting & Statutory Admissibility (BSA 2023 Section 63)**
   - Generates court-ready PDF packages featuring execution logs, evidence hash parity tables, carved frame thumbnails, statutory **BSA 2023 Section 63 Admissibility Certificates**, and itemized **Numbered I-Frame Stills (PNG + SHA-256)** for charge sheet annexures.

---

## Technical Architecture & Tech Stack

Locus is built as a **monorepo standalone desktop application**, running a native Electron frontend wrapper powered by an embedded Python FastAPI engine.

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

| Layer | Technology | Role & Purpose |
| :--- | :--- | :--- |
| **Desktop Wrapper** | **Electron** | Cross-platform desktop shell (Windows, Linux, macOS) with native OS file dialogs |
| **Frontend UI** | **React + TypeScript + Vite** | High-performance SPA with interactive video grid & timeline player |
| **UI Styling** | **Tailwind CSS + ShadcnUI** | Professional dark-mode design system matching enterprise forensic tools |
| **Single Backend** | **FastAPI (Python 3.11+)** | Unified local API, raw sector carving, PyAV stream remuxing, WebSockets |
| **Database** | **SQLite 3** | Zero-configuration single-file database (`forensics.db`) |
| **AI Engine** | **ONNX Runtime (YOLOv8)** | Lightweight, hyper-fast local CPU/GPU object, face, and motion detection |
| **Evidence Hashing**| **Python `hashlib`** | Cryptographic SHA-256 and MD5 chain-of-custody verification |

---

## Team Roles & Responsibilities

| Role # | Position | Primary Responsibilities | Core Stack |
| :---: | :--- | :--- | :--- |
| **1** | **Systems Architect & Team Lead** | Overall system design, database schemas, API contracts, SIH presentation strategy | FastAPI, Architecture, SQLite, Docker |
| **2** | **Full-Stack Developer** | Case intake UI, evidence management APIs, REST/WebSocket integration | React, TypeScript, FastAPI, SQLite |
| **3** | **Python & Systems Forensics Dev** | Binary disk image parsing, OEM header carving, PyAV/FFmpeg stream remuxing | Python, Binary I/O, PyAV, FFmpeg |
| **4** | **AI / Computer Vision Dev** | ONNX Runtime pipeline, YOLOv8 object/face detection, motion heatmaps | Python, ONNX, YOLOv8, OpenCV |
| **5** | **Frontend & UI/UX Specialist** | Multi-camera video sync player, interactive timeline matrix, report UI | React, TypeScript, Canvas, Video APIs |
| **6** | **Docs, QA & Research Specialist** | OEM comparative paper, SOPs, legal hash validation testing, final SIH slides | Technical Writing, QA, Research, SOPs |

---

## Key Differentiators

1. **Cross-Platform Linux & Windows Compatibility:** While commercial tools are locked strictly to Windows, Locus runs natively on Linux (which NTRO and defense agencies prefer).
2. **Zero-Configuration Desktop Executable:** Ships as a single `.exe` or `.AppImage`. No complex server setups or database installations required.
3. **Integrated Computer Vision & Timeline Search:** Combines raw sector carving with instant natural language AI indexing (*"Find person on Channel 2"*).
4. **Lightweight ONNX AI Execution:** Motion-gated architecture runs fast on standard laptop CPUs without requiring expensive $2,000 gaming GPUs.
