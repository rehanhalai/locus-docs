# Step 7 Specification: Timeline Synchronization & Multi-Camera Alignment

*This document outlines the technical architecture, mathematical models, and multi-stream playback engine used to synchronize varying camera timelines into a single, cohesive forensic view.*

---

## 1. The Forensic Clock Problem in CCTV Systems

In multi-camera surveillance forensics, video streams from different channels are almost never recorded with perfectly identical or synchronized timing. Forensic investigators encounter three conflicting sources of temporal truth:

1. **Header Timestamp (DVR RTC):** 32-bit packed datetime embedded in proprietary container (e.g., DHAV). Represents DVR RTC when written to disk.
2. **PTS / DTS (Codec):** Presentation/Decode timestamps in the NAL container stream (e.g., 90kHz timebase). Dictates relative frame cadence & playback rate.
3. **OSD Burned-In Clock:** Visual text burned into the pixels by camera firmware. Visible to humans, but unindexed binary data.

### Real-World Failure Modes Handled:
- **CMOS Battery Failure / Factory Reset:** The DVR clock resets to `2000-01-01 00:00:00`, while the crime occurred on `2026-08-24`.
- **Manual Misconfiguration & Drift:** The DVR RTC was never synced to NTP and runs +14m 23s ahead of true UTC/IST. Over 30 days, cheap crystal oscillators drift by an additional ±3 to 30 seconds.
- **Multi-Device Discrepancies:** Camera 1 (Indoor) clock is +5m 10s. Camera 2 (Outdoor) clock is -2m 45s.
- **Motion-Triggered Recording Gaps:** Channel 1 records 24/7 continuous stream; Channel 2 records only on motion, leaving large timeline voids.
- **Mixed Framerates (VFR / CFR):** Channel 1 captures at 25.0 FPS (40ms interval); Channel 2 captures at 12.5 FPS (80ms interval).

---

## 2. Non-Destructive Forensic Time-Layer Architecture

**Forensic Rule of Preservation:**
Locus **never** alters the original byte payloads, carved MP4 frame timestamps, or the raw `stream_headers` SQLite index. Time alignment is achieved via a **Non-Destructive Calibration Layer** mapped over the physical timeline.

```
[ Raw Forensic Image (.dd) ]                          <-- Unmodified Source Evidence Bytes
                           │
                           ▼
[ Step 5/6: SQLite `stream_headers` Index ]           <-- Parsed Source In-Stream Timestamps
                           │
                           ▼
[ Step 7: `timeline_calibrations` Metadata Layer ]    <-- Non-Destructive Virtual Offsets
                           │
                           ▼
[ Multi-Camera Synchronized Playhead Bus ]            <-- Unified Playback Pipeline
```

---

## 3. Database Schema Extensions

To manage multi-channel alignment, Locus introduces two complementary SQLite tables:

### `camera_channels`
Stores metadata and nominal capture profiles for each discovered channel.
```sql
CREATE TABLE IF NOT EXISTS camera_channels (
    channel_id INTEGER PRIMARY KEY,
    channel_name TEXT NOT NULL,
    source_dvr_id TEXT NOT NULL,
    nominal_fps REAL DEFAULT 25.0,
    total_frames INTEGER NOT NULL,
    earliest_raw_timestamp INTEGER NOT NULL,
    latest_raw_timestamp INTEGER NOT NULL,
    has_audio INTEGER DEFAULT 0
);
```

### `timeline_calibrations`
Maintains the mathematical transformation applied to convert raw device timestamps into synchronized, normalized master time.
```sql
CREATE TABLE IF NOT EXISTS timeline_calibrations (
    calibration_id INTEGER PRIMARY KEY AUTOINCREMENT,
    channel_id INTEGER NOT NULL,
    time_offset_ms INTEGER NOT NULL DEFAULT 0,    -- Static delta: Δt (in ms)
    drift_rate_ppm REAL NOT NULL DEFAULT 0.0,     -- Dynamic drift: α (ppm)
    anchor_raw_timestamp INTEGER,                 -- Base reference source timestamp
    anchor_calibrated_utc INTEGER,                -- Known reference UTC timestamp
    calibration_method TEXT NOT NULL,             -- 'OSD_VISUAL', 'NTP_BASELINE', 'MANUAL'
    calibrated_by TEXT NOT NULL,
    FOREIGN KEY(channel_id) REFERENCES camera_channels(channel_id)
);
```

---

## 4. Mathematical Synchronization Model

For any video frame on Channel $i$ with a raw hardware header timestamp $T_{\text{raw}}$, its **Normalized Calibrated Time** $T_{\text{norm}}$ is computed as:

$$T_{\text{norm}}(i) = T_{\text{raw}}(i) + \Delta t_i + \left( \frac{\alpha_i}{10^6} \right) \cdot (T_{\text{raw}}(i) - T_{\text{anchor}, i})$$

### Master Playhead Seek to Stream Offset
When scrubbing the master UI playhead to real-world time $T_{\text{target}}$, Locus queries the closest raw timestamp for each channel:

$$T_{\text{seek}}(i) = T_{\text{target}} - \Delta t_i$$

```sql
SELECT sector_offset, payload_length, timestamp 
FROM stream_headers
WHERE channel_id = :channel_id AND timestamp <= :seek_timestamp
ORDER BY timestamp DESC LIMIT 1;
```

---

## 5. Multi-Camera Grid Synchronization Engine

To deliver seamless playback of up to 16 channels in a grid, Locus uses a **Master-Clock Worker Architecture**:

1. **Master Time Coordinator:** A precision clock (running at 60 Hz) updates the master timeline position $T_{\text{now}}$. All channels are slave consumers.
2. **VFR/CFR Sampling:** If Channel 1 is 30 FPS and Channel 2 is 10 FPS, Channel 2 holds its decoded frame across 3 ticks of the master loop until $T_{\text{now}}$ crosses its next timestamp. No artificial interpolation.
3. **Recording Gaps:** If a channel wasn't recording at $T_{\text{now}}$, the worker displays a distinct forensic overlay (`[ NO RECORDING AT THIS TIMESTAMP ]`) instead of freezing on a stale frame.
4. **Frame-by-Frame Scrubbing:** Stepping forward 1 frame advances the master clock by the smallest frame delta among all active channels.

---

## 6. Real-World Offset Calibration Workflows

Locus provides three calibration workflows to establish ground-truth synchronization:

1. **Visual OSD Anchor Alignment (Most Common):** The investigator locates a real-world event (e.g., a light switch turning on) captured on multiple cameras, pauses playback, and clicks "Set Time Anchor". Locus calculates inter-camera deltas ($\Delta t$).
2. **Burned-in OSD OCR / Time Baseline:** The investigator inputs the exact time shown on the burned-in digital clock. Locus computes the differential between the raw header timestamp and the visible OSD time.
3. **Multi-DVR External Incident Anchor:** Two unrelated DVRs are anchored to an exact Computer Aided Dispatch (CAD) log / police radio timestamp.

---

## 7. Forensic Integrity & Export

1. **Synchronized Multi-View Split Export:** Locus exports a combined multi-grid MP4 (e.g., 2x2 matrix) composited into a single video frame with a burned-in Master Synchronized UTC banner.
2. **Sidecar Synchronization File (`.sync.json`):** Exported `.mp4` files are accompanied by a forensic descriptor containing the channel file hash, applied offset ($\Delta t$), and calibration rationale.
3. **Audit Trail Documentation:** Every manual calibration adjustment is permanently logged with the investigator's credentials, supporting forensic examination and evidence presentation.

---

## Appendix: Plain English Terminology

If the technical math above seems daunting, here is the simple, real-world translation of how this works:

### The "Three Clocks" Problem
A DVR video actually has three different clocks, and they often disagree:
1. **The Motherboard Clock:** Invisible metadata inside the file. (Often wrong if the DVR battery dies).
2. **The Internal Metronome (PTS):** Tells the video player how fast to play the frames. It doesn't know the real time, just "Tick 1, Tick 2, Tick 3".
3. **The Burned-In Clock:** The physical pixels painted onto the screen showing the date and time.

### The "Sticky Note" Rule (Non-Destructive Layer)
We **cannot** edit the original video files to fix broken timestamps—that is called tampering with evidence, and it will get thrown out of court. 
Instead, Locus leaves the raw video files completely untouched. It just creates a "Sticky Note" in the database that says: *"Camera 2 is exactly 5 minutes too fast."* When playing the video, Locus reads the sticky note and virtually shifts the video on the screen.

### The Math Formula Translated
The formula $T_{\text{norm}}(i) = T_{\text{raw}}(i) + \Delta t_i + \text{Drift}$ is just basic addition:
**True Time = Broken Camera Time + (The Offset) + (The Drift)**
- **Offset ($\Delta t$):** If the camera is 5 minutes fast, the offset is just `-5 minutes`.
- **Drift ($\alpha$):** Cheap DVR clocks run slightly too fast or too slow, losing seconds every week. The formula stretches or shrinks the timeline slightly so the video stays in sync over long recordings.

### The Orchestra Conductor (Master Playhead)
Imagine 16 musicians (16 cameras) trying to play a song together. Without a conductor, it sounds terrible. 
In Locus, the **Master Clock** is the Conductor. It ticks exactly 60 times a second. At every tick, it points to all cameras and says, *"Based on your sticky notes, show me the exact frame you should be at right now."* If a camera is choppy (e.g., 10 FPS), the Conductor tells it to just *hold* its current picture on the screen a little longer so it doesn't fall behind.
