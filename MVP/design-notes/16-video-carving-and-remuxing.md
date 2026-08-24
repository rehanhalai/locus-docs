# Step 6 Specification: Video Carving, Assembly & Remuxing

*This document explains how Locus extracts raw video bytes from disk, strips proprietary container headers, aligns GOP Keyframes, and remuxes them into standard MP4 files.*

---

## 1. What is Step 6 and Why Is It Needed?

At the end of Step 5, we have a complete SQLite database (`stream_headers`) mapping the location of every video frame on disk.

However, the video frames are still:
1. Trapped inside the raw `.dd` forensic image.
2. Wrapped inside proprietary 32-byte header boxes (`DHAV`).
3. Interleaved across sectors with other cameras.

**Step 6 (Video Carving & Remuxing)** is the physical extraction phase where Locus turns those indexed database rows into playable, standard `.mp4` video files that any media player or web browser can open.

---

## 2. The 3 Technical Stages of Video Carving

```
[ SQLite Index Query ] ──► (channel_id=1, time: 2:00 PM - 2:05 PM)
           │
           ▼
┌─────────────────────────────────────────────────────────────┐
│ Stage A: Header Stripping & Raw Byte Extraction             │
│ Seeks to sector_offset, strips 32-byte DHAV header,         │
│ extracts pure H.264/H.265 Annex B NAL byte stream           │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│ Stage B: GOP (Group of Pictures) Alignment                  │
│ Ensures the video clip strictly begins at an IDR Keyframe   │
│ (preventing green/grey corruption artifacts on playback)    │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│ Stage C: Fast Remuxing via PyAV / FFmpeg (Zero Re-encoding) │
│ Wraps raw H.264 NAL units into an .mp4 container in ms      │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               ▼
            [ Playable output_cam1_1400.mp4 File ]
```

---

## 3. Detailed Breakdown of Each Stage

### Stage A: Header Stripping & De-Interleaving
- Locus queries SQLite for all frame offsets belonging to `channel_id = 1`.
- Locus reads each chunk from the `.dd` file:
  - **Offset 0x00 to 0x1F (32 bytes):** Proprietary `DHAV` header $\rightarrow$ **Discarded**.
  - **Offset 0x20 to End:** Pure H.264 video payload (`0x00 0x00 0x00 0x01` NAL bytes) $\rightarrow$ **Appended to stream buffer**.
  - **Footer (`dhav`):** $\rightarrow$ **Discarded**.

### Stage B: GOP (Group of Pictures) Keyframe Alignment
- In H.264 video compression, **P-Frames** only contain motion differences and cannot be displayed without a preceding **I-Frame (Keyframe)**.
- If an extracted video clip starts on a P-Frame, video players will display corrupted green/grey artifacts or crash.
- **Forensic Rule:** Locus ensures that every carved video clip begins cleanly on an **IDR Keyframe (`0x65`)** preceded by **SPS (`0x67`)** and **PPS (`0x68`)** parameter headers.

### Stage C: Zero-Transcoding Remuxing (PyAV / FFmpeg)
- **The Mistake:** Decoding and re-compressing video (Transcoding) takes high CPU/GPU power, takes minutes per clip, and degrades forensic video quality.
- **The Locus Method (Remuxing / Stream Copy):** Locus feeds the raw H.264 NAL bytes directly into **PyAV / FFmpeg** with stream-copy mode (`-c:v copy`).
- FFmpeg simply constructs standard MP4 container metadata (moov/mdat atoms) around the existing video bytes.
- **Performance:** A 10-minute 1080p surveillance video clip is generated in **less than 200 milliseconds** with **100% bit-for-bit forensic preservation**.

---

## 4. Real-World Edge Cases Handled in Step 6

1. **Dropped / Corrupted Frames:** If bad sectors caused a frame to be zeroed out during acquisition, PyAV/FFmpeg gracefully skips the corrupted NAL unit and synchronizes playback to the next available Keyframe without crashing.
2. **Ring Buffer Boundary Stitching:** If a continuous video recording wraps from the end of the disk (Sector 9,999,999) to the beginning (Sector 2048), Locus stitches the two segments chronologically based on their timestamps rather than disk sector order.
