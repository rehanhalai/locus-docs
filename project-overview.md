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

**Locus** bridges this gap by offering a unified, vendor-agnostic software suite. It automates raw sector parsing, deleted video carving, multi-camera timeline synchronization, cryptographic evidence verification (`SHA-256` & `MD5`), and computer vision AI analytics—delivering legally defensible, court-ready forensic reports from a single desktop interface.

---

## Key Features & Core Capabilities

1. **Automated OEM & File System Identification**
   - Automatically detects proprietary partition layouts, magic header signatures, and video codecs from raw disk dumps (`.dd`, `.raw`, `.E01`).

2. **Sector-Level Video Carving & Deleted Recovery**
   - Bypasses corrupted or missing file system indexes. Scans raw drive sectors to carve deleted, overwritten, or damaged H.264/H.265 video stream fragments.

3. **Multi-Camera Synchronized Master Timeline**
   - Calibrates internal DVR clock drifts and synchronizes heterogeneous camera channels into a unified chronological master timeline with multi-grid web playback.

4. **Cryptographic Chain-of-Custody Integrity**
   - Automatically calculates and verifies `SHA-256` and `MD5` hashes at disk ingestion, file carving, extraction, and report export—ensuring strict legal admissibility (ISO/IEC 27037 compliant).

5. **AI-Powered Computer Vision Analytics**
   - Integrated YOLOv8 models process extracted frames to index detected targets (*people, vehicles, objects*), facial bounding boxes, and motion heatmaps—enabling instant timeline filter queries.

6. **Court-Ready Forensic Reporting**
   - Generates audit-compliant PDF/HTML reports featuring execution logs, evidence hash parity tables, carved frame thumbnails, and investigator signatures.

---

## Technical Architecture & Tech Stack

Locus is built as a **monorepo standalone desktop application**, running a native Electron frontend wrapper powered by an embedded Python FastAPI engine.

```
┌───────────────────────────────────────────────────────────┐
│                    ELECTRON SHELL                         │
│   ┌───────────────────────────────────────────────────┐   │
│   │            REACT + SHADCN UI FRONTEND             │   │
│   │   (Investigator Workspace, Timeline, Reports)     │   │
│   └─────────────────────────┬─────────────────────────┘   │
└─────────────────────────────┼─────────────────────────────┘
                              │ HTTP REST / WebSockets (localhost:8000)
                              ▼
┌───────────────────────────────────────────────────────────┐
│                 FASTAPI BACKEND (Python)                  │
│   (Disk I/O, Sector Carving, Stream Processing, AI Jobs)  │
└───────┬─────────────────────┬─────────────────────┬───────┘
        │                     │                     │
        ▼                     ▼                     ▼
┌──────────────┐     ┌─────────────────┐   ┌────────────────┐
│   SQLite     │     │ PyAV / FFmpeg   │   │  ONNX Runtime  │
│(Local DB File│     │(Binary Carving &│   │ (YOLOv8 AI     │
│  metadata)   │     │ Video Remuxing) │   │ Object Search) │
└──────────────┘     └─────────────────┘   └────────────────┘
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
3. **Integrated Computer Vision & Timeline Search:** Combines raw sector carving with instant natural language AI indexing (*"Find red car on Channel 2"*).
4. **Lightweight ONNX AI Execution:** Runs fast on standard laptop CPUs without requiring expensive $2,000 gaming GPUs.
