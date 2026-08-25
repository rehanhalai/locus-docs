# Feature 04: Video Stream Recovery & Carving Engine

**Back to [[MVP/MVP|MVP]]**

---

## 1. Recovery Order & Strategy Hierarchy

Locus enforces a strict, forensically sound order of recovery methods. Generic carving is utilized strictly as a last resort:

```
[Level 1: Filesystem / Index-Aware Extraction]
- Preferred method: Parse surviving vendor index tables to locate contiguous recordings.
    │
    ▼
[Level 2: Vendor-Aware Structure Parsing]
- Parse vendor-specific sector headers (e.g., Dahua DHAV, Hikvision HKFS frame headers).
    │
    ▼
[Level 3: Metadata-Guided Sector Recovery]
- Reassemble sectors matching channel IDs, sequence numbers, and timestamp continuity.
    │
    ▼
[Level 4: Fragment Reconstruction Engine]
- Link orphaned GOP keyframes using NAL unit header signatures and codec parameter sets.
    │
    ▼
[Level 5: Generic Signature Carving (Fallback)]
- Scans raw sector space for H.264/H.265 NAL start codes (0x00000001).
- Used ONLY when filesystem and vendor headers are completely destroyed.
```

> **Limitations of Generic Carving:**  
> Generic carving alone produces high false positive rates, broken streams, out-of-order frames, missing metadata, and channel cross-contamination. Locus explicitly flags generic carved output as `UNVALIDATED_CARVE`.

---

## 2. Explicit Recovery Status Classifications

Every recovery job produces an explicit recovery status record in SQLite:

- **`RECOVERED`:** Complete video clip extracted with intact index/header metadata and clean container remuxing.
- **`PARTIAL`:** Video stream extracted, but with missing initial keyframes or truncated trailing sectors.
- **`FRAGMENTED`:** Discontiguous sector blocks reassembled based on GOP continuity and frame timestamps.
- **`CORRUPTED`:** Video payload contains unreadable sector blocks or decodability errors.
- **`UNRECOVERABLE`:** Underlying disk sectors have been physically overwritten by subsequent recordings. Recovery is impossible.
- **`UNSUPPORTED`:** Sector layout is recognized but codec or container version is currently unsupported.

---

## 3. GOP Alignment & Zero-Transcoding Remuxing

To ensure playable output without altering source evidence bytes:

```text
Raw Sector Bytes ──► Strip Wrapper Header ──► Snap to nearest I-Frame (SPS/PPS)
                                                      │
                                                      ▼
Derived MP4 File ◄── PyAV / FFmpeg Stream Copy ◄── Naked NAL Units (H.264/H.265)
```

- **GOP Snap-Back:** Walking backward to the nearest I-Frame (IDR Keyframe containing Sequence/Picture Parameter Sets) prevents green-screen decoding artifacts.
- **Zero-Transcoding Stream Copy:** Raw compressed bitstream payload bytes are copied directly into standard `.mp4` container wrappers without decoding or re-encoding.

---

## 4. Derived Artifact Provenance Schema (`derived_artifacts`)

Every carved or extracted video clip maintains a strict provenance record:

| Column Name | Data Type | Sample Value | Description |
| :--- | :--- | :--- | :--- |
| `artifact_id` | `TEXT` (PK) | `"ART-2026-904"` | Unique artifact record ID |
| `evidence_id` | `TEXT` (FK) | `"EVD-2026-001"` | Source disk image ID |
| `source_offset` | `INTEGER` | `104857600` | Start byte offset in disk image |
| `source_length` | `INTEGER` | `14857600` | Extracted byte length |
| `channel_id` | `INTEGER` | `2` | Camera channel number |
| `recovery_method`| `TEXT` | `"VENDOR_AWARE_CARVE"` | `INDEX_EXTRACT`, `VENDOR_CARVE`, `GENERIC_CARVE` |
| `parser_version` | `TEXT` | `"DahuaDHAVAdapter v1.2.0"` | Adapter and version used |
| `recovery_status`| `TEXT` | `"RECOVERED"` | Recovery status classification |
| `output_path` | `TEXT` | `"/derived/clip_ch02_001.mp4"`| Local path to derived MP4 clip |
| `sha256_hash` | `TEXT` | `"4f8b91a0c..."` | Cryptographic SHA-256 hash of derived clip |
