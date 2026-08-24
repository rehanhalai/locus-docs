# MASTER PROJECT STATUS & ARCHITECTURAL STATE (LOCUS FORENSICS)

*This file serves as the definitive bridge and master context file across workspaces and conversations.*

---

## 1. Project Identity & Objective
- **Project Name:** Locus Forensics
- **Problem Statement:** SIH 2026 PS 26150 (National Technical Research Organisation - NTRO)
- **Goal:** Multi-Vendor DVR/NVR Forensic Analysis Tool for standardized acquisition, recovery, and analysis of surveillance evidence.
- **Current Scope:** Technical MVP Prototype (legal/BSA 2023 compliance modules deferred; focused purely on technical acquisition, parsing, carving, playback, and analysis).

---

## 2. Completed & Finalized Steps Summary

| Step # | Stage Name | Engine / Tool | Core Technical Decisions | Design Note File |
| :--- | :--- | :--- | :--- | :--- |
| **Step 1 & 2** | Physical Connection & Acquisition | `dc3dd` / Hardware Write-Blocker | Subprocess execution of `dc3dd` with `conv=noerror,sync`, on-the-fly SHA-256/MD5 hashing, bad sector zero-padding, dual input support. | [`06-finalized-input-acquisition-spec.md`](./06-finalized-input-acquisition-spec.md) |
| **Step 3 & 4** | Device & Filesystem Identification | Two-Phase Heuristic Scanner | Phase 1 (Master Superblock: `DHFS`, `HIKBTREE`, `WFS 0.4`, `UVFS`, `HNWL`, `VIGI`). Phase 2 (Sector boundary sampling: `DHAV`, `HIKB`, raw H.264 NAL `0x00000001`). | [`12-vendor-identification-strategy.md`](./12-vendor-identification-strategy.md), [`14-detailed-vendor-signatures-and-phases.md`](./14-detailed-vendor-signatures-and-phases.md) |
| **Target Scope**| The Big 6 Target Vendors | Unified Architecture | 1. Dahua & 2. CP PLUS (Unified DHFS engine), 3. Hikvision (HKFS/B-Tree), 4. Uniview (UNV), 5. Honeywell (Hybrid), 6. TP-Link VIGI + Universal NAL Carver. | [`11-target-vendors-and-filesystem-profiles.md`](./11-target-vendors-and-filesystem-profiles.md), [`13-dahua-cpplus-unified-engine.md`](./13-dahua-cpplus-unified-engine.md) |
| **Step 5** | Sector Metadata Parsing & Indexing | SQLite (`stream_headers`) | Extracts `sector_offset`, `channel_id`, `timestamp`, `payload_length`, `frame_type`. Uses WAL mode (`PRAGMA journal_mode=WAL`) and B-Tree indexing on `(channel_id, timestamp)` with `executemany` batch inserts for sub-millisecond queries. | [`15-sector-metadata-and-header-parsing.md`](./15-sector-metadata-and-header-parsing.md) |
| **Step 6** | Video Carving & Remuxing | PyAV / FFmpeg (Stream Copy) | Strips 32-byte `DHAV` headers, enforces GOP alignment (snapping to IDR Keyframes with SPS/PPS to avoid green glitches), performs zero-transcoding container remuxing (`-c:v copy`) into `.mp4` in milliseconds. | [`16-video-carving-and-remuxing.md`](./16-video-carving-and-remuxing.md), [`17-gop-nal-units-and-remuxing-explained.md`](./17-gop-nal-units-and-remuxing-explained.md) |

---

## 3. Pending Workflow Steps to Discuss & Finalize

- **Step 7:** Timeline Synchronization & Multi-Camera Alignment (Offset calculation, camera grid synchronization).
- **Step 8:** AI-Assisted Analytics & Motion Detection (`DVR-Scan` & `Ultralytics YOLOv8`).
- **Step 9:** Hash Verification & Integrity Export.
- **Step 10:** Case Reporting & GUI Architecture.
