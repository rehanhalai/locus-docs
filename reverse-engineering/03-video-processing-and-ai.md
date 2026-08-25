# 3. Media Remuxing & Secondary AI Triage Pipeline

After forensic parsers extract valid byte ranges from disk image sectors, media processing and analytical triage operations begin.

---

## 1. PyAV / FFmpeg Remuxing Boundaries

Raw video payloads extracted from DVR sector headers (such as Dahua `.dav` or raw elementary streams) cannot be rendered directly in web browsers.

### Zero-Transcoding Remuxing
Locus uses PyAV (Python C-bindings for FFmpeg) in **stream copy** mode (`-c:v copy`):
- **Container Demuxing/Remuxing:** Wraps raw H.264/H.265 bitstream payloads into standard `.mp4` container structures.
- **Zero Transcoding:** Compressed video bitstream payloads (H.264/H.265 NAL units) are preserved without decoding or re-encoding.
- **Derived Artifact Hash:** The newly generated `.mp4` file is a derived artifact with its own cryptographic hash (`derived_artifact_sha256`), distinct from the source disk image hash. Both are tracked in provenance records.
- **Forensic Boundary:** PyAV/FFmpeg is **not** used to parse proprietary DVR filesystems or raw disk layouts. It operates solely on byte streams already validated by forensic parsers.

---

## 2. Secondary AI Triage Pipeline (YOLOv8 via ONNX Runtime)

Video evidence processing extracts 1 frame thumbnail per second for local computer vision analysis:

1. **Frame Sampling:** Extract keyframes from validated `.mp4` clips.
2. **Local Inference:** Pass frames to local ONNX Runtime executing YOLOv8 object detection (`person`, `vehicle`).
3. **Metadata Storage:** Index bounding box coordinates, confidence scores, and frame timestamps into SQLite (`case_meta.db`).
4. **Investigator Triage:** Investigators search and filter events in the React UI and tag detections (`Verified`, `Rejected`, `Unreviewed`).

> **Safety Reminder:** AI detection outputs are candidate suggestions for triage, not primary proof of identity.
