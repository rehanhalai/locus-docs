# 03 — System Architecture & Tech Stack

This document explains how the different parts of Locus are built, how they communicate, and why each technology was chosen.

---

## 1. System Architecture Overview

Locus is built as an **offline-first desktop application**. It combines a modern React user interface running inside an Electron desktop window with a high-performance Python FastAPI backend engine.

```
┌────────────────────────────────────────────────────────────────────────┐
│                        DESKTOP SHELL (Electron)                        │
│  ┌──────────────────────────────────────────────────────────────────┐  │
│  │                    FRONTEND UI (React + Vite)                    │  │
│  │     Investigator Workspace • Multi-Cam Grid • Search Gallery     │  │
│  └──────────────────────────────────┬───────────────────────────────┘  │
└─────────────────────────────────────┼──────────────────────────────────┘
                                      │ HTTP REST & Server-Sent Events (SSE)
                                      │ (http://localhost:8000)
                                      ▼
┌────────────────────────────────────────────────────────────────────────┐
│                     BACKEND ENGINE (FastAPI / Python)                  │
│                                                                        │
│  • `acquisition/` ──► Subprocess runner for bundled `dc3dd`            │
│  • `identification/`► Binary scanner for MBR/GPT & OEM magic bytes     │
│  • `carving/`       ──► Binary `struct.unpack` & PyAV stream remuxing  │
│  • `timeline/`      ──► Master clock & virtual time calibration        │
│  • `analytics/`     ──► OpenCV motion filter + YOLOv8 ONNX detector    │
└─────────────────────────────────────┬──────────────────────────────────┘
                                      │
                                      ▼
                           ┌─────────────────────┐
                           │ SQLite Database     │
                           │ (`forensics.db`)    │
                           └─────────────────────┘
```

---

## 2. Why We Chose This Tech Stack

| Component | Technology | Why We Chose It |
| :--- | :--- | :--- |
| **Desktop Shell** | **Electron** | Provides cross-platform desktop windows (Windows & Linux), OS file pickers, and manages the Python backend process lifecycle. |
| **Frontend UI** | **React + Vite + TypeScript** | Delivers a high-speed, type-safe interface for complex timeline scrubbing, multi-camera video playback grids, and dark-mode styling via Tailwind CSS / ShadcnUI. |
| **Backend Engine** | **FastAPI (Python 3.11+)** | Python is the undisputed standard for binary forensics and AI. FastAPI provides high-speed asynchronous APIs, background job workers, and automatic OpenAPI documentation. |
| **Real-time Streaming**| **Server-Sent Events (SSE)** | Since Locus only needs to push progress telemetry (speed, %, ETA) from server to client, SSE provides a clean, standard HTTP stream without the protocol overhead or connection fragility of WebSockets. |
| **Video Engine** | **PyAV / FFmpeg** | Low-level C-bindings that allow us to unpack proprietary video wrappers and remux raw H.264/H.265 frames into standard `.mp4` using **zero-transcoding** (`-c:v copy`). |
| **AI / CV Engine** | **ONNX Runtime (YOLOv8)** | Instead of requiring users to install 4 GB PyTorch dependencies, we run lightweight, quantized INT8 models locally on any CPU or GPU with zero configuration. |
| **Database** | **SQLite 3 (`forensics.db`)** | Zero-configuration single-file database. It acts as our **Master Sector Index** and immutable chain-of-custody ledger. |
| **Drive Acquisition** | **Bundled `dc3dd` (Static)** | DoD forensic tool that clones disks sector-by-sector, pads bad sectors with null bytes (`conv=noerror,sync`), and computes dual `SHA-256`/`MD5` hashes on-the-fly. Bundled as a standalone static binary with zero host dependencies. |

---

## 3. Communication Protocols Between Frontend and Backend

1. **Standard Commands (HTTP REST):**
   - Triggering actions like `POST /api/acquire/physical`, `POST /api/identify/device`, or querying events `GET /api/search/events`.
2. **Progress Telemetry (Server-Sent Events - SSE):**
   - For long-running operations (disk cloning, deep partition scanning, AI detection), the backend streams live progress frames (`GET /api/identify/events/{job_id}`) over `text/event-stream`.
   - The React frontend connects using the browser's native `new EventSource()` API.
3. **Local Database Synchronization:**
   - All state, case metadata, discovered camera channels, and audit logs are written directly to SQLite (`forensics.db`).

---

*Next up: Read [04-developer-workflow-and-reading-guide.md](./04-developer-workflow-and-reading-guide.md) for the codebase walkthrough and reading roadmap.*
