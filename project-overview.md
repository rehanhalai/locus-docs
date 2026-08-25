# Project Overview: LOCUS

**Project Name:** Locus  
**Tagline:** Forensically Defensible Multi-Vendor DVR/NVR Acquisition Analysis & Reconstruction Platform  
**SIH Problem Statement ID:** 26150  
**Target Organization:** National Technical Research Organisation (NTRO) / Law Enforcement Agencies  
**Theme:** Cybersecurity & Digital Forensics | **Category:** Software  

---

## 1. Executive Summary

**Locus** is a desktop digital forensics and surveillance-storage reconstruction platform designed to solve the critical problem of video evidence fragmentation across heterogeneous CCTV and DVR/NVR systems.

Modern CCTV recorders from manufacturers such as **Dahua, Hikvision, CP Plus, Honeywell, TP-Link, Godrej, Uniview, and Matrix** frequently bypass standard operating system filesystems (like NTFS or EXT4). Instead, they write video streams directly to raw disk sectors using proprietary partition layouts, ring-buffer allocation schemas, custom container headers, and non-standard stream multiplexing. During forensic investigations:
- Commercial OS file tools fail to mount or recognize drive structures.
- Proprietary vendor viewing utilities are often closed-source, Windows-only, unvalidated, and fail on deleted or damaged sectors.
- Generic file carving tools produce fragmented, out-of-order, unplayable, or timestamp-corrupted media files without metadata.
- Clock drift across multiple camera channels distorts multi-angle chronological reconstruction.

**Locus** addresses these challenges through a modular forensic engineering architecture. It ingests pre-acquired forensic disk images (`.dd`, `.raw`, `.img`), performs read-only storage layout detection, executes vendor-aware structure parsing and sector recovery, normalizes timestamps while preserving raw metadata, establishes full artifact provenance, validates stream integrity, and generates structured Forensic Evidence Reports. 

Computer vision AI (YOLOv8 via ONNX Runtime) is incorporated strictly as a **secondary analytical/triage tool** after evidence extraction to help investigators search and filter extracted media.

---

## 2. Central Forensic Safety Principles

> [!IMPORTANT]
> **Primary Rule of Locus Engineering:**  
> **Locus prioritizes forensic correctness over successful-looking output.** A system crash or an explicit `UNKNOWN` classification is infinitely safer than a false positive or a confidently wrong forensic output. A wrong-but-plausible video stream is a major risk in an investigation.

Locus adheres to 12 mandatory forensic principles:

1. **Read-Only Source Preservation:** Never modify source evidence disk images under any operational state.
2. **Upstream Hashing:** Calculate and record cryptographic hashes (`SHA-256`, `MD5`) immediately upon evidence registration before any downstream analysis.
3. **Raw Timestamp Integrity:** Always preserve raw, original source timestamps alongside any normalized UTC timeline representation.
4. **Byte Preservation:** Preserve raw extracted bytes; never silently mutate, alter, or interpolate source payload bytes.
5. **Zero Metadata Invention:** Never invent, extrapolate, or hallucinate missing metadata (such as timestamps, channel IDs, or frame rates).
6. **Strict Parser Scoping:** Never force unknown or malformed sector headers into a vendor parser without validated header/layout signature matching.
7. **No Overwrite Promises:** Never promise or claim video reconstruction after underlying drive sectors have been physically overwritten.
8. **Complete Artifact Provenance:** Every derived media clip or frame must maintain an explicit lineage back to its source image ID, byte offset, sector length, parser version, and processing parameters.
9. **Parser Versioning:** Version all parsing logic and adapters to ensure complete processing history tracking.
10. **Secondary AI Scoping:** AI object and face detection results are probabilistic triage aids, not primary ground truth or proof of identity.
11. **Explicit Uncertainty Reporting:** If layout, metadata, or media structure is ambiguous, report it explicitly as `UNKNOWN`, `PARTIAL`, `AMBIGUOUS`, or `CORRUPTED`.
12. **Reproducibility:** Repeated processing of the same evidence file using identical software versions and configurations must yield identical results.

---

## 3. Product Positioning & Scope

### Core Differentiator
Forensically defensible multi-vendor DVR/NVR acquisition analysis, storage reconstruction, stream validation, timeline normalization, and artifact provenance.

### What AI Does and Does NOT Do in Locus

| AI Capabilities in Locus (Secondary Triage Layer) | What AI CANNOT Do (Forensic Boundaries) |
| :--- | :--- |
| Accelerates investigator search (*"Filter frames containing red vehicles on Channel 1"*). | **Cannot** recover missing source video bytes or fix damaged sectors. |
| Computes probabilistic bounding boxes and confidence scores. | **Cannot** establish legal authenticity or replace cryptographic hashing. |
| Generates motion heatmaps for timeline indexing. | **Cannot** repair incorrect evidence acquisition or corrupted sector headers. |
| Provides structured frame metadata for human verification. | **Cannot** prove personal identity based on object detection alone. |

> [!WARNING]
> All AI analytical findings are clearly designated as **"AI-Generated Analytical Findings"** and require human investigator review (`Verified`, `Rejected`, or `Unreviewed`).

---

## 4. MVP Scope vs. Future Roadmap

Locus clearly delineates its Minimum Viable Product (MVP) boundary from future research phases:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           LOCUS MVP SCOPE                                   │
│  Input: Pre-acquired Forensic Disk Images (.dd, .raw, .img)                 │
│  - Read-Only Image Registration & Dual Hashing (SHA-256 / MD5)              │
│  - Storage Layout & Header Signature Detection                             │
│  - Deeply Validated Adapters (Dahua DHAV, Hikvision HKFS profiles)          │
│  - Intact Recording Extraction & Surviving Deleted Carving Fallback        │
│  - Media Integrity Validation & Raw/Normalized Timeline Engine               │
│  - Cryptographic Provenance Tracking & Forensic Evidence Reporting          │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                       FUTURE / OUTSIDE MVP SCOPE                            │
│  - Physical Hardware Write-Blocker Integration & Direct Disk Imaging        │
│  - E01 / AFF4 Proprietary Evidence Container Ingestion                      │
│  - Specialized Physical Memory (RAM) Analysis                               │
│  - Additional Proprietary Vendor Adapters (Godrej, Matrix, TP-Link profiles)│
└─────────────────────────────────────────────────────────────────────────────┘
```

> **Application-Level Read-Only Analysis vs. Hardware Write Protection:**  
> Opening an evidence image read-only in software prevents application-level write operations. It is **not** equivalent to a hardware write blocker for physical drive acquisition. Physical acquisition remains a separate upstream process.

---

## 5. Non-Legal Disclaimer & Terminology Corrections

Locus strictly avoids uncertified legal claims:
- **No "Court-Ready" or "Legally Admissible" Guarantees:** Legal admissibility is decided solely by courts based on legal standards, chain of custody, and expert testimony.
- **Corrected Terminology:** Locus output is described as a **"Forensic Evidence Report"** designed to **"support forensic examination, provide cryptographic integrity verification, document processing provenance, and enable independent reproducibility."**
- **Standards Alignment:** Locus design aligns with forensic principles in ISO/IEC 27037 (Digital Evidence Handling), but does not claim independent laboratory certification unless validated.

---

## 6. End-to-End System Architecture

Locus uses a decoupled, local desktop architecture where UI, backend API, and core forensic parsing layers are strictly separated:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      ELECTRON DESKTOP WRAPPER                               │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                 REACT + TYPESCRIPT + TAILWIND UI                    │   │
│   │  (Evidence Intake, Hex Viewer, Timeline Grid, Report Inspector)     │   │
│   └──────────────────────────────────┬──────────────────────────────────┘   │
└──────────────────────────────────────┼──────────────────────────────────────┘
                                       │ HTTP REST / WebSockets (localhost:8000)
                                       ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                      FASTAPI BACKEND ENGINE (Python 3.11+)                  │
│                                                                             │
│  ┌───────────────────────┐ ┌───────────────────────┐ ┌───────────────────┐  │
│  │ Evidence Ingestion &   │ │ Storage Layout &      │ │ Vendor Adapters   │  │
│  │ Cryptographic Hashing  │ │ Header Detection      │ │ (Dahua, Hikvision)│  │
│  └───────────┬───────────┘ └───────────┬───────────┘ └─────────┬─────────┘  │
│              │                         │                       │            │
│              ▼                         ▼                       ▼            │
│  ┌───────────────────────┐ ┌───────────────────────┐ ┌───────────────────┐  │
│  │ Video Recovery &      │ │ Media Integrity       │ │ Time Engine &     │  │
│  │ Carving Engine        │ │ Validation Engine     │ │ Timeline Normalizer│
│  └───────────┬───────────┘ └───────────┬───────────┘ └─────────┬─────────┘  │
│              │                         │                       │            │
│              ▼                         ▼                       ▼            │
│  ┌───────────────────────┐ ┌───────────────────────┐ ┌───────────────────┐  │
│  │ Secondary AI Engine   │ │ Provenance & Audit    │ │ Report Generator  │  │
│  │ (ONNX / YOLOv8)       │ │ Logger (SQLite)       │ │ (HTML / PDF)      │  │
│  └───────────────────────┘ └───────────────────────┘ └───────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 7. Technology Rationale & Architectural Responsibility

| Technology | Architectural Responsibility | Rationale for Selection | Technical Limitations & Safeguards |
| :--- | :--- | :--- | :--- |
| **Electron** | Cross-platform desktop window shell & OS dialog handler. | Provides native OS integration (file picker, windowing) across Windows & Linux. | Isolated from backend filesystem operations; UI communicates strictly via localhost REST/WS APIs. |
| **React + TS** | Investigator workspace UI & interactive canvas timeline. | Type-safe UI components for complex multi-camera grid playback & state management. | UI does not perform raw byte parsing; strictly renders data provided by FastAPI engine. |
| **FastAPI** | Local asynchronous REST & WebSocket API controller. | High-performance Python async framework for orchestrating heavy disk I/O and worker pools. | Binds strictly to `127.0.0.1` (localhost) to prevent network exposure. |
| **SQLite 3** | Single-file local case database (`case_meta.db`). | Zero-configuration, transactional local storage for evidence records, audit logs, and provenance. | Database stores metadata, hashes, offsets, and AI tags only—never raw video bytes. |
| **PyAV / FFmpeg**| Container demuxing, remuxing, and frame decoding. | Robust Python bindings to FFmpeg C libraries for standard video stream container handling. | Used *after* forensic parsing identifies valid frame offsets. Not used as a raw proprietary layout parser. |
| **ONNX Runtime** | Secondary CPU/GPU AI inference engine (YOLOv8). | Highly optimized cross-platform C++ runtime for local neural network inference without heavy dependencies. | Inference runs locally; confidence scores are stored as probabilistic metadata. |
| **`hashlib`** | Cryptographic hash computation (`SHA-256` & `MD5`). | Native Python library utilizing optimized C implementations for streaming block-hash calculation. | Uses 64 KB chunked reads to hash multi-terabyte disk images efficiently without RAM exhaustion. |
| **WeasyPrint + Jinja2** | HTML→PDF report rendering. | Pure-Python pipeline for deterministic local Forensic Evidence Report generation (no headless browser required). | No JavaScript execution; no network calls; output is reproducible from identical SQLite data and template version. |

---

## 8. Vendor Support & Adapter Matrix

Vendor support requires laboratory-validated device profiles, not table entries.

| Vendor | Tested Models | Firmware Profiles | Storage / Filesystem Layout | Adapter Name | Carving Status | Validation Status |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Dahua** | NVR4xxx, HCVR5xxx | V3.210+, V4.0+ | Proprietary DHAV index / Raw sector frames | `DahuaDHAVAdapter` | Intact Index + Sector Carve | **Partially Validated** |
| **CP Plus** | Cosmic / Orange series | OEM Dahua derivative | DHAV variant / Custom index header | `CPPlusAdapter` | DHAV fallback | **Researching (Phase 2 — not in MVP)** *(Requires lab data)* |
| **Hikvision**| DS-76xx, DS-77xx | HIK-V3.4+, V4.0+ | HKFS (Hikvision File System) / Raw sector index | `HikvisionHKFSAdapter`| HKFS Sector Index | **Partially Validated** |
| **Honeywell**| HEN series | OEM derivative | Custom container / Standard FAT32 export | `HoneywellAdapter` | Container demux | **Planned** |
| **TP-Link** | VIGI NVR1008H | Firmware V1.x | Standard Linux EXT4 / Proprietary index | `TPLinkAdapter` | Filesystem parse | **Planned** |
| **Godrej** | ESEE series | OEM derivative | Unknown proprietary layout | `GodrejAdapter` | Generic Fallback | **Researching** *(Requires lab data)* |
| **Uniview** | NVR301 series | UNV-V3.x | Custom stream layout | `UniviewAdapter` | Generic Fallback | **Researching** *(Requires lab data)* |
| **Matrix** | SATATYA series | Proprietary firmware | Custom disk partition schema | `MatrixAdapter` | Generic Fallback | **Researching** *(Requires lab data)* |

---

## 9. Performance Targets & Benchmarking Plan

Locus replaces arbitrary performance claims with concrete target metrics and a validation plan:

| Metric Category | Target Performance | Measured Value | Testing Conditions |
| :--- | :--- | :--- | :--- |
| **Evidence Registration & Hashing** | ≥ 150 MB/s (Sequential Read) | *To be benchmarked* | 1 TB raw image file on NVMe SSD storage |
| **Header Signature Scan Rate** | ≥ 300 MB/s | *To be benchmarked* | 512-byte sector-aligned read buffer pool |
| **Stream Remuxing Throughput** | ≥ 50 fps (H.264/H.265 passthrough)| *To be benchmarked* | Direct stream copy without re-encoding |
| **AI Detection Processing** | ≥ 15 fps (640x640 frame input) | *To be benchmarked* | ONNX CPU Execution Provider (Intel i7 / Ryzen 7) |
| **Max RAM Utilization** | ≤ 2.5 GB peak usage | *To be benchmarked* | 4-channel simultaneous processing pool |

> **Note:** The 1 TB test image used for the Evidence Registration & Hashing benchmark reflects the **Recommended Forensic Workstation** specification (500 GB+ NVMe SSD), not the **Minimum Demo Laptop** specification (10 GB free space) listed in `architecture-and-tech-stack.md` Section 5 (Workstation System Requirements). Demo-environment benchmarks will use proportionally smaller test images.

---

## 10. Key Differentiators for Hackathon Evaluation

1. **Forensic Safety First:** Built on explicit uncertainty modeling (`UNKNOWN` / `UNSUPPORTED`) rather than deceptive best-guess outputs.
2. **Transparent Provenance:** Full traceability from derived MP4 clips down to exact disk image sector byte offsets.
3. **Decoupled Architecture:** Clean separation of disk parsing logic from UI and AI components.
4. **Local Execution & Privacy:** 100% offline, local-only processing protecting sensitive investigation data.
5. **Secondary AI Integration:** Pragmatic use of computer vision for investigator triage without overclaiming legal authority.
