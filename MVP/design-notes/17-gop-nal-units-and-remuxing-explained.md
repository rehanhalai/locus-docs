# Deep-Dive Guide: GOP, Frame Types (I/P/B), SPS/PPS, and Zero-Transcoding Remuxing

*This document provides an intuitive, plain-language explanation of video compression mechanics, NAL units, GOP alignment, and how zero-transcoding remuxing works in Locus.*

---

## 1. Why Video Compression Exists (The CCTV Storage Problem)

If a surveillance camera recorded 1080p video as uncompressed full images at 30 frames per second:
- Each frame = ~6 Megabytes
- 1 Second of video = **180 Megabytes**
- 1 Hour of video = **648 Gigabytes**

A 1TB hard drive would fill up in less than 2 hours for just 1 camera!

To solve this, video codecs like **H.264 (AVC)** and **H.265 (HEVC)** do not save full photos 30 times a second. They save one full photo occasionally, and then only record the **tiny motion differences (deltas)** between frames.

---

## 2. The Types of Video Frames: I-Frames vs. P-Frames

Surveillance video is made of two main types of frames:

### A. I-Frame (Intra-Frame / Keyframe)
- **What it is:** A complete, full-resolution image (like a standalone JPEG photo).
- **Independence:** It does NOT depend on any other frame. If you open an I-Frame by itself, you see the entire scene clearly.
- **Size:** Large (e.g., 50 KB to 100 KB).

### B. P-Frame (Predicted Frame / Delta Frame)
- **What it is:** Only stores what **moved or changed** since the last frame (e.g., *"the background didn't move; the car moved 4 pixels to the left"*).
- **Dependence:** It **CANNOT** be displayed on its own. It requires the previous I-Frame to render properly.
- **The Catch:** If you try to play a video starting on a P-Frame, your media player has no background image to work with, resulting in **corrupted green/grey pixel smear**.
- **Size:** Tiny (e.g., 2 KB to 5 KB).

*(Note: B-Frames predict motion both forwards and backwards, but are rarely used in CCTV DVRs to avoid processing delay).*

---

## 3. What is an IDR Keyframe?

- In H.264, a standard I-Frame might still allow future frames to reference frames *before* it.
- An **IDR (Instantaneous Decoder Refresh) Frame** is a special, strict Keyframe that tells the video player:
  > *"Clear all memory of past frames. Nothing after this point will ever look back at anything before this point."*
- **Forensic Rule:** Every carved video clip **must start on an IDR Frame** so it can play independently without needing older disk sectors.

---

## 4. What are SPS and PPS? (The "Blueprint" & "Recipe")

Before a video player can draw an IDR frame on screen, it needs two tiny metadata packets called **Parameter Sets**:

1. **SPS (Sequence Parameter Set - NAL `0x67`):**
   - The **Blueprint**: Tells the player the video resolution (e.g., `1920x1080`), aspect ratio (`16:9`), and framerate (`25 FPS`).
   - Without SPS, the media player doesn't even know how large the window should be!
2. **PPS (Picture Parameter Set - NAL `0x68`):**
   - The **Recipe**: Tells the player the exact mathematical decompression algorithms to use.

---

## 5. What is a GOP (Group of Pictures)?

A **GOP** is a complete, self-contained cluster of video frames that begins with an **IDR Keyframe** and is followed by a sequence of **P-Frames**:

```
┌────────────────────────────────────────────────────────────────────────┐
│                        ONE COMPLETE GOP (1–2 Seconds)                  │
├──────────────┬──────────────┬──────────────┬───────────┬───────────────┤
│     SPS      │     PPS      │ IDR Keyframe │  P-Frame  │    P-Frame    │
│ (Resolution) │ (Decompress) │ (Full Photo) │ (Delta 1) │   (Delta 2)   │
└──────────────┴──────────────┴──────────────┴───────────┴───────────────┘
```

In CCTV systems, a new GOP starts every 1 to 2 seconds (every 25 to 50 frames).

### Why the "GOP Alignment Stage" in Locus is Mandatory:
If an investigator asks for video from `14:02:05` to `14:05:00`:
- If `14:02:05` happens to land on a P-Frame in the middle of a GOP, Locus does not start carving there.
- Locus **snaps back to the nearest preceding IDR Keyframe** (e.g., at `14:02:04`).
- This guarantees that frame #1 of the resulting MP4 has the SPS/PPS headers and full IDR photo, producing a crystal-clear, artifact-free video.

---

## 6. What is "Zero-Transcoding Remuxing"? (The Envelope Analogy)

### The Wrong Way: Transcoding / Re-encoding (Slow & Lossy)
- Decompresses the video into raw pixels in RAM, and then uses a CPU/GPU video encoder to compress it again into an MP4 file.
- **Flaws:** Takes minutes per clip, pegs CPU at 100%, and alters/degrades the original camera pixel evidence.

### The Locus Way: Remuxing / Stream Copy (Instant & 100% Lossless)
Think of the video data as a **letter** and the file format as an **envelope**:
- Inside the CCTV hard drive, the video frames are already compressed in H.264 (the letter). They are just sitting inside a proprietary Dahua/Hikvision "shipping envelope".
- **Remuxing (`-c:v copy` in PyAV/FFmpeg):**
  1. We take the exact unchanged H.264 byte stream out of the proprietary envelope.
  2. We place those exact same bytes into a standard `.mp4` envelope (writing the 1 KB MP4 container header).
- **Speed:** Takes **0.1 seconds** (200ms) for a 10-minute clip!
- **Forensic Integrity:** Not a single pixel is altered. The cryptographic hash of the video payload remains 100% identical to the source disk.
