# Step 5 Specification: Sector-Level Metadata & Header Parsing (The SQLite Master Map)

*This document explains how Locus reads proprietary headers across sectors and builds a high-performance SQLite index of all video frames.*

---

## 1. What is Step 5 and Why Is It Needed?

Now that Locus has identified the vendor (e.g., Dahua or Hikvision), it faces a major hurdle:

**The Interleaving & Scrambling Problem:**
CCTV hard drives do not store video files sequentially. A 4-camera DVR records all 4 cameras simultaneously, writing small chunks from Camera 1, then Camera 2, then Camera 3, then Camera 4, scrambled across sectors in a circular buffer.

If we tried to play this raw data directly, the video player would crash because frames from 4 different cameras would collide every few sectors.

**The Solution in Step 5:**
Locus scans the disk, decodes every proprietary frame header, and builds an in-memory / SQLite **"Master Map"** that catalogs:
- Exactly where every video frame starts (Byte Offset)
- Which camera channel it belongs to (Channel 1, 2, 3, etc.)
- Exactly what date and time it was recorded
- Exactly how many bytes long the frame is

---

## 2. Anatomy of a Dahua Frame Header

For Dahua (`DHAV`), every video chunk starts with a 32-byte binary header:

```
Offset 0x00: [ 4 Bytes ] -> Magic: "DHAV" (0x44 0x48 0x41 0x56)
Offset 0x04: [ 1 Byte  ] -> Channel ID (e.g., 0x00 = Cam 1, 0x01 = Cam 2)
Offset 0x05: [ 1 Byte  ] -> Frame Type (0xFD = I-Frame Keyframe, 0xFC = P-Frame, 0xF0 = Audio)
Offset 0x06: [ 2 Bytes ] -> Sequence / Frame Number
Offset 0x08: [ 4 Bytes ] -> Payload Length (Size of the H.264 video bytes that follow)
Offset 0x0C: [ 4 Bytes ] -> Timestamp (32-bit packed Date/Time)
Offset 0x10: [ 16 Bytes] -> Reserved / Checksum / Video Resolution flags
```

---

## 3. High-Speed SQLite Database Schema (`stream_headers`)

Locus creates a dedicated, ultra-fast SQLite index database for the case:

```sql
CREATE TABLE IF NOT EXISTS stream_headers (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    sector_offset INTEGER NOT NULL,      -- Exact byte offset on the .dd disk
    channel_id INTEGER NOT NULL,         -- Camera number (1 to 64)
    frame_type TEXT NOT NULL,            -- 'I' (Keyframe) or 'P' (Delta)
    payload_length INTEGER NOT NULL,     -- Exact byte size of the raw video frame
    timestamp INTEGER NOT NULL,          -- Unix Epoch timestamp (e.g., 1700000000)
    is_fragmented INTEGER DEFAULT 0      -- Flag for sector-split recovery
);

-- B-Tree indexes for instantaneous multi-camera queries
CREATE INDEX IF NOT EXISTS idx_channel_time ON stream_headers (channel_id, timestamp);
CREATE INDEX IF NOT EXISTS idx_sector ON stream_headers (sector_offset);
```

---

## 4. Performance Engineering: Handling Millions of Records

A 1TB CCTV drive can contain **10,000,000+ individual video frames**.
If a program executes standard single `INSERT INTO` queries in a loop, SQLite will collapse, taking 45+ minutes to write the index.

### Locus Performance Optimization Rules:
1. **WAL Mode (Write-Ahead Logging):** Enables asynchronous, lock-free disk writes (`PRAGMA journal_mode = WAL;`).
2. **Batch Inserts (`executemany`):** Locus buffers 50,000 parsed headers in RAM memory and commits them in a single batch transaction every 1 second.
3. **Scan Speed:** Allows Locus to index an entire 1TB disk in **under 2 to 3 minutes**.

---

## 5. What This Unlocks for the Next Step (Carving)

Once the `stream_headers` table is populated:
- Carving out **"Camera 2 between 2:00 PM and 2:15 PM"** requires a single low-latency SQL query:
  ```sql
  SELECT sector_offset, payload_length 
  FROM stream_headers 
  WHERE channel_id = 2 
    AND timestamp BETWEEN 1700000000 AND 1700000900 
  ORDER BY timestamp ASC;
  ```
- Locus simply seeks to those exact sector offsets, reads the exact byte lengths, and hands them to FFmpeg to generate the MP4 clip!
