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

---

## Tech Stack Table

| Component             | Technology                | Purpose                                                                          |
| :-------------------- | :------------------------ | :------------------------------------------------------------------------------- |
| **Disk Acquisition**  | `dc3dd` Engine            | Live physical bit-stream cloning, bad sector defense, and on-the-fly hashing     |
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
| **Storage** | 5 GB free SSD space *(Note: For MVP `.dd` file ingestion, host must have free space > drive size)* | High-speed NVMe SSD (100 GB+ free space) |
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

1. **Acquisition & Intake:** `dc3dd` clones physical drive or investigator loads `.dd` image → Baseline SHA-256/MD5 hashes generated.
2. **Identification:** Scans disk for OEM magic signatures (Dahua `DHAV`, Hikvision).
3. **Sector Mapping:** Unpacks proprietary binary headers to build a Master Sector Map in SQLite.
4. **Carving & Remuxing:** Background FastAPI thread reads sector blocks and uses PyAV/FFmpeg to remux raw video to `.mp4` using zero-transcoding.
5. **Timeline Sync:** Normalizes camera timestamps onto a unified 60 Hz master clock bus.
6. **AI Indexing:** ONNX Runtime processes extracted frames using YOLOv8, indexing detected objects with timestamps.
7. **Evidence Search:** UI queries SQLite allowing investigators to filter via natural language parameters.
8. **Hash Integrity:** Hashes re-verified at export to generate cryptographic audit sidecar (`.sync.json`).
9. **UI & Reporting:** React frontend generates Forensic Evidence PDF report.
