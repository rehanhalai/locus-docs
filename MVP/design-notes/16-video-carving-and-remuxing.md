# Step 6 Specification: Video Carving, Assembly & Remuxing

**Back to [[MVP/MVP|MVP]]**

---

## 1. What is Step 6 and Why Is It Needed?

At the end of Step 5, Locus maintains a Master Sector Map (`stream_headers`) cataloging frame offsets, channels, timestamps, and frame types across the evidence image.

However, the video data is still:
1. Located inside the raw `.dd` forensic image file.
2. Encapsulated inside proprietary container structures (such as validated DHAV wrappers).
3. Interleaved across sectors with recordings from other camera channels.

**Step 6 (Video Carving & Stream-Preserving Remuxing)** is the extraction phase where Locus recovers validated byte streams and packages them into derived `.mp4` video artifacts for forensic review and timeline presentation.

---

## 2. The 3 Technical Stages of Video Carving

```
[ SQLite Master Map Query ] ──► (channel_id=1, time: T1 - T2)
           │
           ▼
┌─────────────────────────────────────────────────────────────┐
│ Stage A: Header Stripping & Bitstream Extraction            │
│ Seeks to sector_offset, removes container header wrapper,   │
│ extracts raw H.264/H.265 Annex B NAL byte stream            │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│ Stage B: GOP (Group of Pictures) Keyframe Alignment         │
│ Attempts alignment to an available valid IDR/keyframe       │
│ to ensure clean decoding without smear artifacts            │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│ Stage C: Stream-Preserving Remuxing via PyAV / FFmpeg       │
│ Packages raw NAL units into standard .mp4 container         │
│ in stream-copy mode (-c:v copy) without re-encoding         │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               ▼
             [ Derived Artifact: output_cam1.mp4 ]
```

---

## 3. Detailed Breakdown of Each Stage

### Stage A: Header Stripping & De-Interleaving
- Locus queries SQLite for all frame offsets belonging to a specific channel.
- Locus reads each chunk from the evidence image:
  - **Header Wrapper:** Stripped based on the validated vendor profile (e.g., for specific validated DHAV profiles, observed video chunks use a 32-byte header structure; header layout is model, firmware, and profile dependent and requires validation).
  - **Video Payload:** Extracted as raw H.264/H.265 Annex B bitstream bytes (`0x00 0x00 0x00 0x01` NAL units).
  - **Footer:** Discarded if present in the validated profile.

### Stage B: GOP (Group of Pictures) Keyframe Alignment
- In H.264/H.265 compression, P-Frames store motion differentials relative to prior reference frames and cannot be decoded independently.
- **Forensic Alignment Rule:** Locus attempts to align a recovered clip to an available valid IDR/keyframe when sufficient source data exists (preceded by valid SPS and PPS parameter sets). If no suitable keyframe is available, the recovered artifact may be classified as `PARTIAL`, `FRAGMENTED`, or `CORRUPTED` according to validation results.

### Stage C: Stream-Preserving Remuxing (PyAV / FFmpeg)
- **Forensic Tool Boundary:** PyAV/FFmpeg is **not** used to parse proprietary DVR filesystems or raw storage images. The forensic adapter handles storage layout parsing; PyAV/FFmpeg operates solely on extracted, validated elementary streams.
- **Stream-Copy Mode (`-c:v copy`):** When stream-copy remuxing is possible, the recovered compressed elementary-stream payload is preserved without decoding or re-encoding.
- **Derived Artifact Lineage:** The resulting `.mp4` file is a **derived artifact** containing newly constructed container metadata (moov/mdat atoms). It is independently hashed (`artifact_sha256`) and linked to the parent evidence image (`source_master_sha256`) in SQLite and `.sync.json` provenance sidecars.
- **Performance:** Remuxing performance is hardware-, codec-, container-, and stream-dependent and will be measured during validation benchmarking. *(Target benchmark: to be measured).*

---

## 4. Real-World Edge Cases Handled in Step 6

1. **Dropped / Corrupted Sectors:** If bad sectors or null-padded blocks are encountered, the parser records the missing sector range in `audit_log`, logs the recovery status as `PARTIAL` or `CORRUPTED`, and attempts stream resynchronization at the next valid IDR keyframe.
2. **Ring Buffer Boundary Stitching:** If a continuous recording wraps from the end of physical disk sectors to the beginning of the circular partition, Locus orders and stitches segments chronologically based on parsed header timestamps rather than physical sector order.
