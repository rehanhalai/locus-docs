# System Architecture, Technology Justification & Security Model

**Back to [[locus]]**

---

## 1. End-to-End System Architecture

Locus is built as a modular desktop application featuring strict decoupling between UI rendering, local REST API orchestration, raw forensic stream parsing, stream validation, and analytical AI processing.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      ELECTRON DESKTOP WRAPPER                               │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │               REACT + TYPESCRIPT + TAILWIND UI                      │   │
│   │  (Evidence Workspace, Sector Hex Viewer, Sync Timeline, Reports)    │   │
│   └──────────────────────────────────┬──────────────────────────────────┘   │
└──────────────────────────────────────┼──────────────────────────────────────┘
                                       │ Local REST APIs / WebSockets (127.0.0.1:8000)
                                       ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                    FASTAPI FORENSIC BACKEND ENGINE                          │
│                                                                             │
│  Layer 1: Evidence Ingestion & Metadata Registry                            │
│  - Image file registration (.dd, .raw, .img) & SHA-256 / MD5 hashing        │
│                                                                             │
│  Layer 2: Device Identification & Layout Scanner                            │
│  - Sector-aligned magic header scanner & partition table analyzer           │
│                                                                             │
│  Layer 3: Storage & Filesystem Analysis Engine                              │
│  - MBR / GPT / Proprietary partition layout mapping                         │
│                                                                             │
│  Layer 4: Vendor Adapter Pipeline                                           │
│  - Modular adapters: DahuaDHAVAdapter, HikvisionHKFSAdapter, etc.           │
│                                                                             │
│  Layer 5: Video Recovery & Carving Engine                                   │
│  - Filesystem index parser + Metadata-guided sector recovery                │
│                                                                             │
│  Layer 6: Media Validation Engine                                           │
│  - Container, decoder, frame continuity, and timestamp validation           │
│                                                                             │
│  Layer 7: Time Engine & Multi-Camera Timeline Normalizer                    │
│  - Raw timestamp preservation, clock offset calibration, UTC timeline       │
│                                                                             │
│  Layer 8: Secondary AI Analytics Engine                                     │
│  - ONNX Runtime (YOLOv8) frame detection (object/face/motion triage)       │
│                                                                             │
│  Layer 9: Custody, Provenance & Audit Logger                                │
│  - SQLite transactional metadata & complete processing event logging        │
│                                                                             │
│  Layer 10: Forensic Evidence Report Generator                               │
│  - Structured HTML/PDF export with hash parity tables & provenance trace     │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Comprehensive Technology Justification

Every technology selection in Locus is justified by specific forensic, performance, and security requirements.

| Component | Technology | Rationale & Problem Solved | Forensic Safeguards & Limitations |
| :--- | :--- | :--- | :--- |
| **Desktop Wrapper** | **Electron** | Provides cross-platform desktop shell (Windows, Linux) with native OS file dialogs and window management. | Isolated process sandbox; UI layer cannot issue raw disk or file modification calls directly. |
| **Frontend UI** | **React 18 + TypeScript** | Type-safe UI framework for complex multi-camera video synchronization matrix and timeline UI. | UI is strictly presentation/control layer. Never performs raw byte parsing or metadata extraction. |
| **UI Styling** | **Tailwind CSS + ShadcnUI** | Professional, accessible dark-mode UI system matching standard forensic workstation aesthetics. | Custom components provide clean hex viewer and timeline overlays. |
| **Backend Engine** | **FastAPI (Python 3.11+)** | High-performance Python async web framework for local REST endpoints, WebSocket event streaming, and process orchestration. | Binds strictly to loopback interface `127.0.0.1`. Network access is disabled. |
| **Local Database**| **SQLite 3 (`case_meta.db`)** | Zero-configuration, transactional single-file database for storing case metadata, file indexes, hashes, and audit events. | Database stores metadata, offsets, and AI tags only—never raw evidence payload bytes. |
| **Media Processing**| **PyAV + FFmpeg** | C-optimized Python bindings for demuxing, remuxing, and frame decoding of recovered video payloads. | Operates *only* on byte ranges validated by forensic parsers. Not used as a proprietary filesystem reader. |
| **AI Inference** | **ONNX Runtime** | High-speed C++ neural network inference engine running YOLOv8 ONNX models locally without GPU dependencies. | Execution outputs probabilistic candidate tags. Results are labeled secondary and require human review. |
| **Hashing Engine** | **Python `hashlib`** | Native C-backed implementation of SHA-256 and MD5 cryptographic block hashing. | Uses 64 KB block streaming to compute hashes on multi-terabyte disk images without memory bloat. |
| **Report Generator**| **WeasyPrint + Jinja2** | Pure-Python HTML→PDF rendering pipeline for deterministic local Forensic Evidence Report generation. | No JavaScript execution; no network calls; output is byte-for-byte reproducible from identical SQLite records and template versions. |

---

## 3. Security & Evidence Safeguards Model

Locus implements strict security controls to prevent evidence contamination and local vulnerability exploitation:

1. **Local-Only API Binding:**  
   The FastAPI backend binds strictly to `127.0.0.1:8000`. No external network sockets are opened, preventing unauthorized remote access to evidence metadata.
2. **Read-Only File Handles:**  
   All disk image I/O operations are opened using strict read-only flags (`rb` mode in Python / `O_RDONLY` at OS level). Software write operations to the source file handle are programmatically prohibited.
3. **Electron IPC Isolation:**  
   Electron `contextIsolation` is enabled and `nodeIntegration` is disabled. Renderer processes interact with backend APIs via restricted IPC channels.
4. **Isolated Temporary Working Directory:**  
   Derived media clips (`.mp4` remuxes) and extracted frames are written to a designated workspace directory (`./workspace/cases/<case_id>/derived/`). Source evidence images are never altered in place.
5. **Audit Logging & Transaction Tracking:**  
   All API requests, parser selections, parameter changes, and export actions are logged with ISO-8601 UTC timestamps into `audit_log` in SQLite.
6. **Session Cleanup Policy:**  
   Temporary frame caches used for AI inference can be purged on demand or upon case closure without affecting evidence provenance records.

---

## 4. Processing Pipeline Execution Flow

```
Raw Evidence Image (.dd / .raw)
    │
    ▼
[Step 1: Ingestion & Baseline Hashing]
- Compute SHA-256 and MD5 hashes using 64 KB chunk streaming.
- Register Evidence ID and file metadata in SQLite database.
    │
    ▼
[Step 2: Device & Storage Layout Detection]
- Read sector 0 (MBR) / sector 1 (GPT) partition tables.
- Perform sector-aligned signature scan across first 100 MB.
- Calculate candidate vendor layout confidence (Dahua DHAV, Hikvision HKFS, Unknown).
    │
    ▼
[Step 3: Vendor Adapter Parsing & Recovery]
- IF validated adapter exists -> Execute filesystem index parsing.
- IF index corrupted/missing -> Execute metadata-guided sector carving.
- IF vendor unknown -> Fallback to generic signature carving & flag as UNKNOWN layout.
    │
    ▼
[Step 4: Media Integrity & Timestamp Validation]
- Parse container demux structures; verify stream continuity.
- Preserve raw timestamp; interpret timezone/clock drift; generate UTC normalized timeline.
- Assign validation status: VALID, PARTIAL, CORRUPTED, or UNSUPPORTED.
    │
    ▼
[Step 5: Secondary AI Triage Engine (Optional)]
- Sample valid frames at configurable intervals (e.g., 1 frame/sec).
- Run ONNX Runtime (YOLOv8) object/face/motion detection.
- Save bounding boxes and confidence scores linked to frame timestamps.
    │
    ▼
[Step 6: Investigator Workspace & Report Export]
- Multi-camera synchronized grid playback in React UI.
- Human review of AI findings (Verified / Rejected).
- Generate PDF/HTML Forensic Evidence Report with full provenance traceability.
```

---

## 5. Workstation System Requirements

| Hardware / OS Metric | Minimum Requirements (Demo Laptop) | Recommended Requirements (Forensic Workstation) |
| :--- | :--- | :--- |
| **Operating System** | Windows 10/11 (64-bit) / Ubuntu 20.04+ LTS | Windows 11 (64-bit) / Ubuntu 22.04 LTS |
| **Processor (CPU)** | 4-Core Intel Core i5 / AMD Ryzen 5 (2.5 GHz+) | 8+ Core Intel Core i7/i9 or AMD Ryzen 7/9 |
| **System Memory (RAM)**| 8 GB RAM (4 GB available for process pool) | 16 GB – 32 GB DDR4/DDR5 RAM |
| **Primary Storage** | 10 GB free space on SSD | 500 GB+ High-Speed NVMe SSD for derived caches |
| **GPU / Acceleration**| Integrated Graphics (CPU ONNX Execution Provider)| NVIDIA RTX 3060+ (CUDA Acceleration for AI triage) |

> **Note:** Performance benchmarks in `project-overview.md` Section 9 (e.g., the 1 TB Evidence Registration & Hashing target) are measured against the **Recommended Forensic Workstation** spec above. The Minimum Demo Laptop spec (10 GB free space) is sufficient for functional demonstration but not for full-scale benchmarking.
