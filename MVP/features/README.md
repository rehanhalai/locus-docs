# Locus Core MVP Feature Specifications (Single Source of Truth)

> [!IMPORTANT]
> **Authoritative Specification Directory (SSOT):**  
> This directory (`MVP/features/`) is the **Single Source of Truth** for the Locus desktop forensic analysis platform.  
> Every feature specification here defines the functional contracts, inputs/outputs, error handling, and database schemas for the system.
>
> * If you want the deep technical research, binary format rationale, and academic citations, see [`MVP/design-notes/`](../design-notes/).
> * If you want the low-level FastAPI endpoints and SQLite queries, see [`MVP/development/backend/`](../development/backend/).

---

## The End-to-End Forensic Processing Pipeline

The Locus backend executes an 8-stage automated pipeline (+ reporting) from physical disk seizure to courtroom export:

```text
[01. Physical Acquisition] ──► [02. Device ID] ──► [03. Sector Header Parsing]
                                                            │
                                                            ▼
[06. Local AI Analytics] ◄── [05. Timeline Sync] ◄── [04. Video Carving & Remux]
          │
          ▼
[07. Evidence Search] ──────► [08. Hash Verification & Export] ──► [09. BSA 2023 Reporting]
```

---

## Master Feature Index

| # | Feature Directory | Specification Document | Core Responsibility | Status |
| :--- | :--- | :--- | :--- | :--- |
| **01** | [`disk-imaging/`](./disk-imaging/) | [`disk-imaging.md`](./disk-imaging/disk-imaging.md) | Live physical drive acquisition via embedded `dc3dd`, bad sector defense (`conv=noerror,sync`), and dual SHA-256/MD5 baseline hashing. | **Complete** |
| **02** | [`device-identification/`](./device-identification/) | [`device-identification.md`](./device-identification/device-identification.md) | Automatic OEM & magic byte signature detection across 8 major DVR/NVR vendors (`DHAV`, `HKFS`, `WFS`, etc.). | **Complete** |
| **03** | [`filesystem-parsing/`](./filesystem-parsing/) | [`filesystem-parsing.md`](./filesystem-parsing/filesystem-parsing.md) | Binary header decoding via `struct.unpack()`, sector mapping, and constructing the Master Sector Map in SQLite (`stream_headers`). | **Complete** |
| **04** | [`video-carving/`](./video-carving/) | [`video-carving.md`](./video-carving/video-carving.md) | Sector-level carving of raw H.264/H.265 frames, GOP snapping to nearest I-Frames, and zero-transcode remuxing (`-c copy`). | **Complete** |
| **05** | [`timeline-sync/`](./timeline-sync/) | [`timeline-sync.md`](./timeline-sync/timeline-sync.md) | Multi-camera timestamp normalization, 60 Hz master clock bus, and non-destructive virtual calibration layers (`timeline_calibrations`). | **Complete** |
| **06** | [`ai-analytics/`](./ai-analytics/) | [`ai-analytics.md`](./ai-analytics/ai-analytics.md) | Local YOLOv8 object detection (persons, vehicles) and OpenCV MOG2 motion void indexing via ONNX Runtime. | **Complete** |
| **07** | [`evidence-search/`](./evidence-search/) | [`evidence-search.md`](./evidence-search/evidence-search.md) | High-speed SQLite query layer for parameterized filtering (camera + time + object + confidence) and thumbnail gallery deep-linking. | **Complete** |
| **08** | [`evidence-hashing/`](./evidence-hashing/) | [`evidence-hashing.md`](./evidence-hashing/evidence-hashing.md) | Sliced clip hash verification, `.sync.json` audit trail sidecar generation, and on-demand whole-disk re-verification. | **Complete** |
| **09** | [`forensic-reporting/`](./forensic-reporting/) | [`forensic-reporting.md`](./forensic-reporting/forensic-reporting.md) | Courtroom-ready PDF reports, statutory **BSA 2023 Section 63 Admissibility Certificates**, and numbered I-frame still exhibits. | **Complete** |
