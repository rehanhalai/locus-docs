# 02 — Core Features & Real-World Analogies

This document explains the **8 core backend features of Locus** in plain English using real-world analogies so you can understand the purpose and mechanics of each module without getting lost in technical jargon.

---

## The End-to-End Pipeline Overview

```
[1. Acquisition] ──► [2. Device ID] ──► [3. Header Parsing] ──► [4. Video Carving]
                                                                        │
                                                                        ▼
[8. Integrity Export] ◄── [7. Evidence Search] ◄── [6. Local AI] ◄── [5. Timeline Sync]
```

---

## Step 1: Physical Acquisition & Case Ingestion

### The Real-World Problem:
We cannot perform our investigation directly on the physical crime scene hard drive because the disk might be physically worn, and working on it directly risks corrupting evidence.

### The Analogy (The Hyper-Speed Photocopier & Fingerprint):
> Imagine you find an ancient, fragile diary at a crime scene. Before anyone reads it, you place it into a specialized photocopier that makes a bit-for-bit, page-for-page replica (`.raw` / `.dd` forensic image). 
> 
> If a page has a smudge or physical tear (a *bad sector*), the machine doesn't crash or rip the book—it smoothly inserts a clean blank page placeholder (`conv=noerror,sync`) to preserve the exact page count. As each page is scanned, it takes an unforgeable digital fingerprint (`SHA-256` and `MD5`) to prove the copy is 100% genuine.

### What Locus Does:
- Clones physical block devices (`/dev/sdb`) using our bundled `dc3dd` engine into a `.dd` file.
- Or locks an existing `.dd` file in strict read-only mode and calculates dual baseline hashes.

---

## Step 2: Device & Manufacturer Identification

### The Real-World Problem:
We have a raw digital file of 1 billion bytes. Which DVR company manufactured it (Dahua, Hikvision, CP Plus, Honeywell)?

### The Analogy (The Book Cover & Language Detector):
> You open the book and look at the cover or the first few words on Page 1. If you see the magic letters `DHFS` or `DHAV`, you know the book is written in Dahua. If you see `HKFS` or `WFS`, you know it is Hikvision.

### What Locus Does:
- Scans the initial disk sectors for manufacturer "magic signatures" (`DHFS`, `HKFS`, `UVFS`, `VIGI`, `0x55AA` MBR).
- Detects the OEM brand, partition boundaries, and camera channel count (e.g., 8-channel vs 16-channel DVR).

---

## Step 3: Sector Header Parsing & Mapping

### The Real-World Problem:
16 cameras record simultaneously, meaning video frames from Camera 1, Camera 3, and Camera 8 are scattered and intermixed across millions of 512-byte disk sectors.

### The Analogy (The Postal Shipping Labels):
> Imagine a warehouse with 10,000 packages dumped in a chaotic pile. Fortunately, every package has a tiny 32-byte shipping label pasted on it saying:
> *"Package for Camera 2 | Recorded: 14:02:15 UTC | Size: 64 KB | Keyframe"*.
> 
> You go through the warehouse, read every shipping label, and write a master directory list (a *Master Sector Map* in SQLite) that records where every package belongs.

### What Locus Does:
- Scans disk sectors using Python `struct.unpack()`, decodes binary C-struct headers, and builds the `stream_headers` table in SQLite so the system knows where each camera stream starts and ends.

---

## Step 4: Sector Video Carving & Stream Remuxing

### The Real-World Problem:
Video frames on the hard drive are trapped inside proprietary vendor wrappers (like Dahua's `DHAV` container) that no standard media player (VLC, Chrome, Windows Media Player) can play.

### The Analogy (The Envelope Swapper):
> Someone sent you an important letter, but it's sealed inside a weird, heavy steel box (`DHAV`). 
> 
> Video carving carefully opens the box, lifts out the untouched original letter (the raw H.264 video bytes) without altering a single word, and places it into a standard paper envelope (`.mp4`) that any media player in the world can open.

### What Locus Does:
- Reads sector offsets from the Master Sector Map, snaps to the nearest Keyframe (I-Frame/IDR) to avoid green screen glitches, strips the proprietary headers, and remuxes raw frames into standard `.mp4` files using **zero-transcoding** (`-c:v copy` via PyAV/FFmpeg).

---

## Step 5: Multi-Camera Timeline Synchronization

### The Real-World Problem:
DVR internal clocks are almost always wrong. Camera 1 might think it is 2:00 PM, while Camera 2 thinks it is 2:05 PM. If you hit play on both, they are completely out of sync.

### The Analogy (The Orchestra Conductor & The Sticky Notes):
> You have 16 musicians trying to play a song together, but 3 of them started 5 seconds too early. You cannot edit their original sheet music (that would be tampering with evidence).
> 
> Instead, you stick a **Sticky Note** on Camera 2's stand saying: *"Play 5 minutes slower."* The **Master Conductor (a 60 Hz master clock)** waves the baton, ensuring all 16 cameras display the exact same real-world second on screen.

### What Locus Does:
- Creates non-destructive virtual calibration layers in SQLite (`timeline_calibrations`).
- Allows investigators to anchor common visual events (e.g., a lightning flash or car door closing) to align all camera playback grids in real time.

---

## Step 6: Local AI Video Analytics

### The Real-World Problem:
Watching 480 camera-days of video manually would take weeks of human labor and cause eye fatigue.

### The Analogy (The 1,000-Page Book Index & Motion Filter):
> If you have a 1,000-page book and need to find every mention of a "Red Car," you don't read every sentence. You flip to the **Index** at the back, find "Red Car: Page 45, 102, 890", and go straight there.
> 
> Before reading, you also have a smart helper who throws away all the blank pages (*Motion Void Filter*), saving you 80% of the reading effort!

### What Locus Does:
- **Phase A (OpenCV MOG2):** Detects motion voids (dead nighttime footage) and tags them as skip zones.
- **Phase B (YOLOv8 via ONNX Runtime):** Runs 100% locally and offline on the investigator's CPU/GPU to detect persons and vehicles, indexing event timestamps into SQLite.

---

## Step 7: Evidence Search & Event Filtering

### The Real-World Problem:
How does an investigator find the exact 3 seconds a suspect appeared among 100,000 indexed AI events?

### The Analogy (The Google Search Bar for CCTV):
> You type: *"Camera 2 + Person + Between 2:00 PM and 4:00 PM + Confidence > 80%"*.
> In less than a second, you get a photo gallery of matching thumbnails. Clicking any thumbnail instantly jumps the video player to that exact moment.

### What Locus Does:
- Executes fast parameterized SQLite queries against the `ai_analytics_events` table and renders clickable search galleries and timeline heatmap markers.

---

## Step 8: Cryptographic Hash Verification & Export

### The Real-World Problem:
When the video clip is brought to court, defense attorneys will ask: *"How do we know this video wasn't altered or generated with AI?"*

### The Analogy (The ATM Digital Receipt):
> When you withdraw cash from an ATM, the machine gives you a paper receipt listing the time, transaction ID, bank account number, and remaining balance.
> 
> Locus acts like a secure ATM for video: it exports the clip using zero-transcoding, calculates its cryptographic hash, and attaches a permanent **Digital Receipt (`.sync.json`)** that links the exported clip and its physical disk sector coordinates directly back to the original seized hard drive’s master hash.

### What Locus Does:
- Performs stream copy export, generates SHA-256/MD5 artifact hashes, writes immutable records to `audit_logs`, and outputs the legal provenance sidecar (`.sync.json`).

---

*Next up: Read [03-architecture-and-tech-stack.md](./03-architecture-and-tech-stack.md) to understand how the frontend, backend, and database collaborate.*
