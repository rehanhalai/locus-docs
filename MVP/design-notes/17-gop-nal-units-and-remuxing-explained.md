# Deep-Dive Guide: GOP, Frame Types (I/P/B), SPS/PPS, and Stream-Preserving Remuxing

**Back to [[MVP/MVP|MVP]]**

---

## 1. Why Video Compression Exists (The CCTV Storage Problem)

If a surveillance camera recorded 1080p video as uncompressed full images at 30 frames per second:
- Each frame = ~6 Megabytes
- 1 Second of video = **180 Megabytes**
- 1 Hour of video = **648 Gigabytes**

A 1TB hard drive would fill up in less than 2 hours for just 1 camera.

To solve this, video codecs like **H.264 (AVC)** and **H.265 (HEVC)** do not save full photos 30 times a second. They save one full photo periodically (Keyframe), and then only record the **motion differences (deltas)** between frames.

---

## 2. The Types of Video Frames: I-Frames vs. P-Frames

Surveillance video streams are predominantly composed of two frame types:

### A. I-Frame (Intra-Frame / Keyframe)
- **What it is:** A complete, standalone image (analogous to a JPEG image).
- **Independence:** It does NOT depend on prior or future frames to render.
- **Size:** Large (e.g., 50 KB to 100 KB).

### B. P-Frame (Predicted Frame / Delta Frame)
- **What it is:** Encodes only what changed relative to the previous reference frame.
- **Dependence:** It **cannot** be rendered on its own. It requires the preceding reference frame.
- **Artifact Consequence:** If playback commences on a P-Frame without an antecedent reference frame, media players produce decoding artifacts (macroblocking, green/grey pixel smear).
- **Size:** Compact (e.g., 2 KB to 5 KB).

*(Note: B-Frames predict motion bi-directionally, but are rarely used in real-time CCTV DVRs to minimize hardware latency).*

---

## 3. IDR Keyframes and Parameter Sets

### What is an IDR Keyframe?
- In H.264/H.265, an **IDR (Instantaneous Decoder Refresh) Frame** is a strict keyframe boundary signaling the decoder to flush all previous reference frame buffers.
- **Forensic Alignment Principle:** Locus attempts to align a recovered clip to an available valid IDR/keyframe when sufficient source data exists. If no suitable keyframe is available, the recovered artifact may be classified as `PARTIAL`, `FRAGMENTED`, or `CORRUPTED` according to validation results.

### What are SPS and PPS?
Before a decoder renders an IDR frame, it requires stream parameter headers:
1. **SPS (Sequence Parameter Set - NAL `0x67` / `0x27`):** Defines video resolution (e.g., `1920x1080`), aspect ratio, profile, and framerate.
2. **PPS (Picture Parameter Set - NAL `0x68` / `0x28`):** Defines entropy coding modes and slice group parameters.

---

## 4. What is a GOP (Group of Pictures)?

A **GOP** is a self-contained sequence of frames beginning with an **IDR Keyframe** (with SPS/PPS) followed by predicted **P-Frames**:

```
┌────────────────────────────────────────────────────────────────────────┐
│                        ONE COMPLETE GOP (1–2 Seconds)                  │
├──────────────┬──────────────┬──────────────┬───────────┬───────────────┤
│     SPS      │     PPS      │ IDR Keyframe │  P-Frame  │    P-Frame    │
│ (Resolution) │ (Parameters) │ (Full Photo) │ (Delta 1) │   (Delta 2)   │
└──────────────┴──────────────┴──────────────┴───────────┴───────────────┘
```

### The GOP Alignment Objective:
If an investigator requests extraction for time range `14:02:05` to `14:05:00`:
- If `14:02:05` lands on a P-Frame mid-GOP, Locus attempts to **snap backward to the nearest preceding valid IDR Keyframe** (e.g., at `14:02:04`).
- This ensures that frame #1 of the extracted stream contains the required SPS/PPS headers and reference intra-frame, enabling clean decoding when source sectors are intact.

---

## 5. Stream-Preserving Remuxing (The Container Analogy)

### Transcoding / Re-encoding (Lossy & Resource-Heavy)
- Decompresses video into raw pixel arrays in memory, then invokes a secondary encoder to re-compress into MP4.
- **Forensic Flaw:** Modifies original pixel data, incurs generational compression loss, alters bitstream hashes, and consumes heavy CPU/GPU resources.

### Locus Method: Stream-Preserving Remuxing (Stream Copy)
Think of the video data as a **letter** and the file format as an **envelope**:
- On the DVR storage image, compressed H.264/H.265 frames (the letter) reside within proprietary vendor sector wrappers.
- **Stream Copy (`-c:v copy` in PyAV/FFmpeg):**
  1. The forensic parser extracts the unmodified compressed NAL byte stream.
  2. PyAV/FFmpeg places those exact compressed bytes into a standard `.mp4` container wrapper (constructing moov/mdat container atoms).
- **Forensic Distinction:**
  - When stream-copy remuxing is possible, the recovered compressed elementary-stream payload is preserved without re-encoding.
  - The resulting `.mp4` remains a **derived artifact** with its own distinct cryptographic hash (`artifact_sha256`), independently recorded and linked to the source evidence (`source_master_sha256`) via provenance records.
- **Performance:** Remuxing performance is hardware-, codec-, container-, and stream-dependent and will be measured during validation benchmarking. *(Target benchmark: to be measured).*
