# System Architecture & Tech Stack (Electron Desktop App)

**Back to [[locus]]**

---

## Core Architecture

**Locus** is built as a standalone desktop forensic application combining a native Electron window with a single embedded Python engine (`FastAPI`).

- **Desktop Shell:** Electron (Node.js runtime + Chromium) providing native windowing, cross-platform UI rendering (Windows/Linux/macOS), OS file pickers, and child process management.
- **Frontend UI:** React + TypeScript + Vite + Tailwind CSS / ShadcnUI (Investigator workspace, multi-camera playback, timeline matrix, court report generator).
- **Single Backend API:** FastAPI (Python 3.11+) running as an embedded background process handling binary sector carving, video stream decoding, and REST/WebSocket endpoints.
- **Media & AI Engine:** PyAV + FFmpeg for raw stream remuxing and ONNX Runtime for lightweight YOLOv8 object/face/motion detection.
- **Database:** Local SQLite 3 database file (`forensics.db`) for zero-configuration, instant case indexing.

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

---

## Tech Stack Table

| Component             | Technology                | Purpose                                                                          |
| :-------------------- | :------------------------ | :------------------------------------------------------------------------------- |
| **Desktop Wrapper**   | Electron                  | Standalone desktop container, cross-platform Chromium rendering, OS file dialogs |
| **Frontend UI**       | React + TypeScript + Vite | Dashboard UI, multi-camera timeline player, evidence tables, report preview      |
| **UI Styling**        | Tailwind CSS + ShadcnUI   | Professional dark-mode design system & components                                |
| **Backend Engine**    | FastAPI (Python 3.11+)    | Single unified backend API, binary sector carving, async job queues              |
| **Video Processing**  | PyAV + FFmpeg             | Raw stream remuxing, proprietary header decoding, MP4 conversion                 |
| **AI / CV Analytics** | ONNX Runtime (YOLOv8)     | Hyper-fast local CPU/GPU object, face, and motion detection                      |
| **Database**          | SQLite 3                  | Zero-configuration single-file database (`forensics.db`)                         |
| **Hashing**           | Python `hashlib`          | SHA-256 and MD5 chain-of-custody verification                                    |

---

## 💻 System Requirements

| Metric | Minimum (Hackathon Demo & Laptops) | Recommended (Forensic Workstation) |
| :--- | :--- | :--- |
| **OS** | Windows 10/11 (64-bit) / Ubuntu 20.04+ / macOS 11+ | Windows 11 (64-bit) / Ubuntu 22.04 LTS |
| **CPU** | Quad-core Intel Core i5 / AMD Ryzen 5 / Apple M1 | 8-core Intel i7/i9 or AMD Ryzen 7/9 |
| **RAM** | 8 GB RAM (4 GB free memory) | 16 GB – 32 GB RAM |
| **Storage** | 5 GB free SSD space | High-speed NVMe SSD (100 GB+ free space) |
| **GPU** | Integrated Graphics (CPU execution via ONNX Runtime) | NVIDIA GTX 1660 / RTX 3060+ (CUDA Accelerated) |

---

## Monorepo Directory Structure

```text
locus/
├── desktop/                  # Electron Shell (main.js, preload.js)
├── frontend/                 # React UI (src/components, pages, App.tsx)
├── backend/                  # FastAPI Python Backend (app/api, carving, media, ai, db)
├── shared/                   # Sample test raw disk dumps (.dd)
└── package.json              # Monorepo root runner
```

---

## Processing Pipeline

1. **Ingestion & Hashing:** User selects raw disk image (`.dd`) via native desktop file picker → FastAPI calculates SHA-256 / MD5 baseline hash.
2. **Parsing & Carving:** Background FastAPI thread reads sector blocks, matches OEM headers (Dahua `DHAV`, Hikvision raw layouts), and carves video streams.
3. **Stream Remuxing:** PyAV / FFmpeg remuxes raw carved video frames into web-playable `.mp4` files without altering original evidence.
4. **AI Indexing:** ONNX Runtime processes extracted frames using YOLOv8, indexing detected objects/faces with timestamps in SQLite.
5. **UI Rendering:** React frontend receives WebSocket updates, populating the multi-camera timeline player with AI search tags and court-ready PDF reports.
