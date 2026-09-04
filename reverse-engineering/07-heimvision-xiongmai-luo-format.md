# 07 - HeimVision & Xiongmai (XM) `luo ` Container Reverse Engineering

## Overview
* **Target Hardware**: HeimVision K9604-W, Xiongmai (XM) NVRs, Jooan, BESDER, Hiseeu.
* **Storage Structure**: GPT Partitioning with a 144 GB primary FAT32 partition.
* **File System Organization**: Pre-allocated 8,388,608-byte (8.00 MB) `FILE0000.DAT` files stored under sequential directories (`DIR00001`, `DIR00002`...).

---

## 32-Byte `luo ` Binary Header Format

Every active recording file begins with a 32-byte binary header:

```
Offset  Length  Type       Field Name      Description
0x00    4       char[4]    magic           b"luo " (0x6C 0x75 0x6F 0x20)
0x04    4       uint32_le  start_time      Recording Start (Unix Timestamp UTC)
0x08    4       uint32_le  end_time        Recording End (Unix Timestamp UTC)
0x0C    4       uint32_le  c1_offset       Byte offset for Camera 1 Stream
0x10    4       uint32_le  c2_offset       Byte offset for Camera 2 Stream
0x14    4       uint32_le  c3_offset       Byte offset for Camera 3 Stream
0x18    4       uint32_le  c4_offset       Byte offset for Camera 4 Stream
0x1C    4       uint32_le  reserved        Padding / flags (0x00000000)
```

---

## 4-Channel Multiplexing & Stream Demuxing

The 4 camera channels are multiplexed sequentially within each 8 MB file block:
* **Camera 1 Payload**: `dat[c1_offset : c2_offset]`
* **Camera 2 Payload**: `dat[c2_offset : c3_offset]`
* **Camera 3 Payload**: `dat[c3_offset : c4_offset]`
* **Camera 4 Payload**: `dat[c4_offset : file_size]`

### Video Codec
* **Streams**: Elementary H.265 / HEVC Main Profile.
* **Resolution / Framerate**: 1920x1080 Full HD @ 25 fps.
* **Remuxing Strategy**: Zero-transcode elementary stream copy into `.mp4` containers (`ffmpeg -f hevc -i pipe:0 -c:v copy -tag:v hvc1`).

---

## Performance & Optimization
* **Direct FAT32 Cluster Lookup**: Directly traverses cluster directory tables to locate recording files without requiring a 150 GB sector-by-sector scan.
* **Carving Speed**: Extracts 4 synchronous camera streams across all days of recording in under **3 seconds**.
