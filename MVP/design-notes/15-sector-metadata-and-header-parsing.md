# Step 5 Specification: Sector-Level Metadata & Header Parsing (The SQLite Master Map)

**Back to [[MVP/MVP|MVP]]**

---

## 1. Objective of Sector-Level Metadata Parsing

Once candidate vendor profiles are registered, Locus parses proprietary frame headers to resolve multi-camera interleaving:

**The Interleaving & Circular Buffer Problem:**
Multi-channel CCTV recorders write streams simultaneously, interleaving data blocks from Camera 1, Camera 2, Camera 3, and Camera 4 across sectors in a circular buffer. Attempting to decode raw sector ranges linearly results in decoder failure and stream corruption.

**The Solution:**
Locus decodes proprietary frame headers and constructs an in-memory and transactional SQLite **Master Sector Map** recording:
- Frame Byte Offset (Start of video payload)
- Channel ID (Camera 1, 2, 3, etc.)
- Raw Timestamp (Source header timestamp)
- Payload Byte Length (Exact compressed video stream length)
- Frame Type (I-Frame / Keyframe vs P-Frame / Delta)

---

## 2. Exemplary Header Profile: Validated Dahua DHAV Structure

> [!NOTE]
> **Profile-Specific Header Structure:**  
> Header layouts vary across vendor models and firmware revisions. The structure below represents an observed, partially validated profile for **Dahua NVR4xxx / HCVR5xxx** series (`DHAV` wrapper):

```
Vendor:             Dahua Technology
Model Series:       NVR4xxx / HCVR5xxx
Observed Profile:   DHAV 32-Byte Sector Header
Validation Status:  Partially Validated (Lab test images)
Known Limitations:  Requires 512-byte sector alignment; unvalidated on 4K native sector images.

Offset 0x00: [ 4 Bytes ] -> Magic: "DHAV" (0x44 0x48 0x41 0x56)
Offset 0x04: [ 1 Byte  ] -> Channel ID (e.g., 0x00 = Channel 1, 0x01 = Channel 2)
Offset 0x05: [ 1 Byte  ] -> Frame Type (0xFD = I-Frame Keyframe, 0xFC = P-Frame, 0xF0 = Audio)
Offset 0x06: [ 2 Bytes ] -> Sequence / Frame Number
Offset 0x08: [ 4 Bytes ] -> Payload Length (Size of compressed video bitstream)
Offset 0x0C: [ 4 Bytes ] -> Packed Timestamp (Date/Time representation)
Offset 0x10: [ 16 Bytes] -> Reserved / Flags / Checksum
```

> **CP Plus Consideration:** CP Plus devices are parsed via `CPPlusAdapter`. Only when a CP Plus image matches a validated DHAV profile with verified checksums is the DHAV parsing pipeline utilized.

---

## 3. SQLite Database Schema (`stream_headers`)

```sql
CREATE TABLE IF NOT EXISTS stream_headers (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    evidence_id TEXT NOT NULL,           -- Source evidence image ID
    sector_offset INTEGER NOT NULL,      -- Exact byte offset in image file
    channel_id INTEGER NOT NULL,         -- Camera channel number
    frame_type TEXT NOT NULL,            -- 'I' (Keyframe) or 'P' (Delta)
    payload_length INTEGER NOT NULL,     -- Exact byte size of video payload
    raw_timestamp INTEGER NOT NULL,      -- Unmodified source header timestamp
    is_fragmented INTEGER DEFAULT 0,     -- Flag for sector-split boundary
    FOREIGN KEY(evidence_id) REFERENCES evidence_sources(id)
);

-- B-Tree indexes for multi-camera queries
CREATE INDEX IF NOT EXISTS idx_channel_time ON stream_headers (channel_id, raw_timestamp);
CREATE INDEX IF NOT EXISTS idx_sector ON stream_headers (sector_offset);
```

---

## 4. Performance Architecture & Target Throughput

Surveillance storage images can contain millions of individual frame records.
- **WAL Mode (Write-Ahead Logging):** Enables asynchronous disk writes (`PRAGMA journal_mode = WAL;`).
- **Batch Inserts (`executemany`):** Buffers 50,000 parsed headers in RAM before committing transactional batches.
- **Performance Classification:** Target processing architecture designed to maximize sequential I/O throughput. *(Formal multi-terabyte benchmark to be measured on standardized datasets).*

---

## 5. Handoff to Video Carving Engine

Once the `stream_headers` table is populated:
- Retrieving candidate frames for **"Channel 2 between T1 and T2"** executes a parameterized SQL query:
  ```sql
  SELECT sector_offset, payload_length, frame_type, raw_timestamp 
  FROM stream_headers 
  WHERE channel_id = 2 
    AND raw_timestamp BETWEEN :start_ts AND :end_ts 
  ORDER BY raw_timestamp ASC;
  ```
- The carving engine seeks to the specified byte offsets, extracts the raw NAL payloads, validates GOP alignment, and performs a stream-preserving remux into derived `.mp4` artifacts.
