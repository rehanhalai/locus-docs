# Locus Architecture Design Notes & Research Archive

> [!IMPORTANT]
> **Documentation Roles & Hierarchy:**
> * **Authoritative Specifications (SSOT):** If you are looking for the locked, final feature definitions, go to [`MVP/features/`](../features/). That is the Single Source of Truth for what Locus builds.
> * **Code Implementation Blueprints:** For low-level FastAPI backend endpoints, database models, and UI wireframes, go to [`MVP/development/`](../development/).
> * **This Directory (`MVP/design-notes/`):** Contains the **engineering rationale, binary format research, deep mathematical proofs, and external forensic citations** that justify why Locus was designed the way it is.

---

## 1. Traceability Matrix: Design Notes to Core Features

The 22 numbered design notes map directly into the official core features:

| Core Feature (in `MVP/features/`) | Relevant Design Notes | Focus / Content in Design Notes |
| :--- | :--- | :--- |
| **01. Physical Acquisition**<br>([`disk-imaging.md`](../features/disk-imaging/disk-imaging.md)) | [Note 01](./01-acquisition-and-parsing.md), [Note 02](./02-mvp-scope-and-ingestion.md), [Note 03](./03-acquisition-and-imaging.md), [Note 04](./04-open-source-integrations.md), [Note 05](./05-user-flow-and-imaging-decisions.md), [Note 06](./06-finalized-input-acquisition-spec.md) | Dual-path acquisition architecture (`dc3dd` live physical drive cloning with bad sector defense + forensic image ingestion) and baseline dual hashing. |
| **02. Device Identification**<br>([`device-identification.md`](../features/device-identification/device-identification.md)) | [Note 07](./07-device-and-filesystem-identification.md), [Note 08](./08-mbr-and-partition-table-analysis.md), [Note 09](./09-sector-aligned-scanning-explained.md), [Note 10](./10-signature-reality-and-two-tier-detection.md), [Note 11](./11-target-vendors-and-filesystem-profiles.md), [Note 12](./12-vendor-identification-strategy.md), [Note 13](./13-dahua-cpplus-unified-engine.md), [Note 14](./14-detailed-vendor-signatures-and-phases.md) | Deep analysis of MBR/GPT offsets, sector-aligned scanning, 2-tier magic signature detection, and unifying Dahua/CP Plus (`DHFS`/`DHAV`). |
| **03. FS Header Parsing**<br>([`filesystem-parsing.md`](../features/filesystem-parsing/filesystem-parsing.md)) | [Note 15](./15-sector-metadata-and-header-parsing.md) | Unpacking 32-byte binary headers with Python `struct.unpack`, false-positive defense, and building the Master Sector Map. |
| **04. Video Carving**<br>([`video-carving.md`](../features/video-carving/video-carving.md)) | [Note 16](./16-video-carving-and-remuxing.md), [Note 17](./17-gop-nal-units-and-remuxing-explained.md) | Byte stream carving, NAL unit parsing, GOP snapping to nearest I-Frames, and zero-transcode remuxing (`-c copy`) via PyAV/FFmpeg. |
| **05. Timeline Sync**<br>([`timeline-sync.md`](../features/timeline-sync/timeline-sync.md)) | [Note 18](./18-timeline-synchronization-and-alignment.md) | Multi-camera timestamp normalization, 60 Hz master clock bus, and non-destructive virtual calibration layers. |
| **06. AI Analytics**<br>([`ai-analytics.md`](../features/ai-analytics/ai-analytics.md)) | [Note 19](./19-ai-analytics-and-motion.md) | OpenCV MOG2 motion void detection (skipping dead space) and local YOLOv8 ONNX Runtime object inference. |
| **07. Evidence Search**<br>([`evidence-search.md`](../features/evidence-search/evidence-search.md)) | *(Covered directly in feature spec)* | Sub-second SQLite indexed querying, multi-parameter filtering, and video deep-linking. |
| **08. Evidence Hashing**<br>([`evidence-hashing.md`](../features/evidence-hashing/evidence-hashing.md)) | [Note 20](./20-hash-verification-and-export.md) | Baseline dual-hash preservation, whole-disk re-verification, and `.sync.json` provenance sidecar generation. |
| **Forensic Standards & Citations** | [Note 21](./21-authentic-vendor-references-and-citations.md) | Academic papers (MDPI 2025), patents, open-source decoders, and authentic NIST CFReDS DVR benchmark images. |
| **Indian Forensic Reality Check** | [Note 22](./22-forensic-reality-check-and-mvp-fumbles.md) | Indian State FSL protocols, BSA 2023 Section 63 admissibility, *Tomaso Bruno* gap defense, and IST clock drift realities. |

---

## 2. Directory File Index

- [`01-acquisition-and-parsing.md`](./01-acquisition-and-parsing.md) — Core architectural decisions on acquisition modalities, MBR scanning, header validation, and ring buffers.
- [`02-mvp-scope-and-ingestion.md`](./02-mvp-scope-and-ingestion.md) — *Early scratchpad:* defining boundaries of the MVP ingestion pipeline.
- [`03-acquisition-and-imaging.md`](./03-acquisition-and-imaging.md) — *Early scratchpad:* write-blocking trade-offs and hashing strategies.
- [`04-open-source-integrations.md`](./04-open-source-integrations.md) — Evaluating open-source tools (`dc3dd`, `pytsk3`, `PyAV`, `FFmpeg`).
- [`05-user-flow-and-imaging-decisions.md`](./05-user-flow-and-imaging-decisions.md) — UI flow considerations for case intake and drive mounting.
- [`06-finalized-input-acquisition-spec.md`](./06-finalized-input-acquisition-spec.md) — **Locked Spec:** The finalized `dc3dd` execution model for live drives.
- [`07-device-and-filesystem-identification.md`](./07-device-and-filesystem-identification.md) — Device signature discovery rules.
- [`08-mbr-and-partition-table-analysis.md`](./08-mbr-and-partition-table-analysis.md) — Sector 0 analysis, MBR vs GPT vs raw unpartitioned layouts.
- [`09-sector-aligned-scanning-explained.md`](./09-sector-aligned-scanning-explained.md) — Why scanning must be aligned to 512-byte / 4096-byte boundaries.
- [`10-signature-reality-and-two-tier-detection.md`](./10-signature-reality-and-two-tier-detection.md) — Two-tier detection: fast header check + secondary payload validation.
- [`11-target-vendors-and-filesystem-profiles.md`](./11-target-vendors-and-filesystem-profiles.md) — Profile breakdowns for Dahua, Hikvision, CP Plus, WFS.
- [`12-vendor-identification-strategy.md`](./12-vendor-identification-strategy.md) — Fast-pass vs deep-carve heuristic fallback logic.
- [`13-dahua-cpplus-unified-engine.md`](./13-dahua-cpplus-unified-engine.md) — Architectural unification of Dahua DHFS and CP Plus OEM rebrands.
- [`14-detailed-vendor-signatures-and-phases.md`](./14-detailed-vendor-signatures-and-phases.md) — Exhaustive table of hex magic bytes and offsets across 8 OEMs.
- [`15-sector-metadata-and-header-parsing.md`](./15-sector-metadata-and-header-parsing.md) — Binary unpacking routines and SQLite `stream_headers` schema.
- [`16-video-carving-and-remuxing.md`](./16-video-carving-and-remuxing.md) — Slicing sector blocks into `.mp4` containers.
- [`17-gop-nal-units-and-remuxing-explained.md`](./17-gop-nal-units-and-remuxing-explained.md) — Technical primer on H.264 NAL units (SPS, PPS, IDR, non-IDR).
- [`18-timeline-synchronization-and-alignment.md`](./18-timeline-synchronization-and-alignment.md) — The 60 Hz master clock bus and virtual calibration algebra.
- [`19-ai-analytics-and-motion.md`](./19-ai-analytics-and-motion.md) — Motion void detection algorithms and local YOLOv8 ONNX pipeline.
- [`20-hash-verification-and-export.md`](./20-hash-verification-and-export.md) — Export slice hashing and `.sync.json` audit trail spec.
- [`21-authentic-vendor-references-and-citations.md`](./21-authentic-vendor-references-and-citations.md) — Academic papers, patents, and NIST CFReDS datasets.
- [`22-forensic-reality-check-and-mvp-fumbles.md`](./22-forensic-reality-check-and-mvp-fumbles.md) — Reality check: Indian FSL standards, BSA 2023 Section 63, and clock drift.
- [`workflow roadmap.md`](./workflow%20roadmap.md) — 10-step investigative lifecycle.
