# Step 4.6 Detailed Guide: Vendor Signature Patterns & Detection Phases

*This document explains the signature patterns of each of the 6 major vendors and details exactly how Phase 1 (Master Probe) and Phase 2 (Frame Probe) work in plain, simple language.*

---

## 1. Understanding the Two Phases (Simple Analogy)

Imagine you are trying to identify a book written in a foreign language:
- **Phase 1 (The Book Title / Fast Path):** You look at the front cover or the first page to read the title. If the title says *"Hikvision Volume"* or *"Dahua DHFS"*, you immediately know the language in less than a second.
- **Phase 2 (Reading Random Sentences / Deep Path):** If the front cover is torn off or burned (corrupted index), you open the book and read random sentences inside. If every paragraph starts with `"DHAV"`, or if you recognize standard video sentences (`0x00000001` NAL units), you still know what the book is!

---

## 2. Detailed Breakdown of Each Vendor

---

### Vendor 1 & 2: Dahua Technology & CP PLUS

#### A. The Real-World Signature Pattern:
Dahua and CP PLUS wrap every video frame inside a proprietary **DHAV "shipping box"**:
- **Header Magic (Start of Frame):** `0x44 0x48 0x41 0x56` (ASCII: **`DHAV`**)
- **Footer Magic (End of Frame):** `0x64 0x68 0x61 0x76` (ASCII: **`dhav`**)
- **Internal Payload:** Inside the box is standard H.264/H.265 compressed video.

```
┌─────────────────┬───────────┬──────────────┬───────────────────┬─────────────────┐
│  "DHAV" Header  │ Channel ID│  Timestamp   │ Frame Video Bytes │  "dhav" Footer  │
│    (4 bytes)    │ (1 byte)  │  (4 bytes)   │   (Raw H.264)     │    (4 bytes)    │
└─────────────────┴───────────┴──────────────┴───────────────────┴─────────────────┘
```

#### B. How Locus Detects Dahua / CP PLUS:
- **Phase 1 (Fast Path):** Locus reads the first sector of the video partition. If it sees the filesystem identifier **`DHFS`** (`0x44 0x48 0x46 0x53`), it confirms Dahua/CP PLUS.
- **Phase 2 (Deep Path):** If the filesystem table is damaged, Locus scans sector boundaries. When it sees `DHAV` repeating every few thousand bytes followed by matching `dhav` footers, it confirms Dahua/CP PLUS.

---

### Vendor 3: Hikvision (and EZVIZ)

#### A. The Real-World Signature Pattern:
Hikvision works like a library with a master catalog index called **`HKFS` / `HIKBTREE`**:
- Hikvision does **not** put the word "Hikvision" on every frame.
- Instead, it writes a master **B+ Tree Index Table** at the beginning of the partition.
- **Master Signatures:** `0x48 0x4B 0x46 0x53` (**`HKFS`**) or `0x48 0x49 0x4B 0x42` (**`HIKB`**) or **`HIKBTREE`**.
- The video sectors themselves contain raw video clusters (often grouped in 2MB or 16MB blocks).

#### B. How Locus Detects Hikvision:
- **Phase 1 (Fast Path):** Locus reads the start of the video partition (e.g., Sector 2048). If it finds the `HIKBTREE` or `HKFS` master table, it confirms Hikvision. Locus then reads the tree to instantly get all camera recordings and dates.
- **Phase 2 (Deep Path):** If the master index is destroyed, Locus looks for Hikvision cluster boundary headers (`HIKB` blocks) or falls back to raw H.264 carving.

---

### Vendor 4: Uniview (UNV)

#### A. The Real-World Signature Pattern:
Uniview uses structured Linux partitions with proprietary volume headers:
- **Master Signatures:** Look for **`UBIT`**, **`UVFS`**, or **`UNV`** in the volume superblock descriptors.
- **Video Storage:** Streams are stored in pre-allocated sector blocks wrapping standard Transport Stream (TS) or H.264/H.265 elementary streams.

#### B. How Locus Detects Uniview:
- **Phase 1 (Fast Path):** Checks the GPT/MBR partition volume labels and superblock headers for `UNV` / `UVFS`.
- **Phase 2 (Deep Path):** Scans for Uniview block segment markers or raw H.264 start codes.

---

### Vendor 5: Honeywell Security

#### A. The Real-World Signature Pattern:
Honeywell enterprise NVRs use a hybrid architecture:
- Partition 1 is often a standard Linux EXT4 volume for system configuration.
- Partition 2 is a large proprietary storage pool tagged with **`HNWL`** or utilizing Dahua-derived `DHFS` headers (as Honeywell OEMs certain hardware lines).

#### B. How Locus Detects Honeywell:
- **Phase 1 (Fast Path):** Reads partition labels for `HNWL` volume tags. If found, initializes Honeywell profile.
- **Phase 2 (Deep Path):** Scans video payload. If Dahua-style `DHAV` headers or raw H.264 NALs are detected, parses accordingly.

---

### Vendor 6: TP-Link (VIGI / Tapo)

#### A. The Real-World Signature Pattern:
TP-Link's commercial VIGI NVR line uses a custom segmented circular buffer:
- **Master Signatures:** Superblock tags **`VIGI`** or `TP-LINK` in the GPT partition header.
- **Video Chunks:** Uses fixed-size circular recording chunks with internal timestamp indexes.

#### B. How Locus Detects TP-Link:
- **Phase 1 (Fast Path):** Reads GPT volume headers for `VIGI` or `TP-LINK` identifiers.
- **Phase 2 (Deep Path):** Detects VIGI chunk boundary headers and raw H.264/H.265 NAL units.

---

## 3. The Universal Safety Net (Generic H.264 NAL Carver)

What if a hard drive is heavily damaged, burned, formatted, or from an unknown budget DVR?

**The Raw Video Codec Reality:**
Regardless of who manufactured the DVR, almost all modern surveillance cameras compress video using **H.264 (AVC)** or **H.265 (HEVC)**.

In H.264, every video frame *must* start with a 4-byte Start Code:
* **`0x00 0x00 0x00 0x01`**

Immediately following those 4 bytes is the **NAL Unit Type Byte**:
* `0x67` $\rightarrow$ **SPS (Sequence Parameter Set):** Contains video resolution (e.g., 1920x1080) and framerate (e.g., 30 FPS).
* `0x68` $\rightarrow$ **PPS (Picture Parameter Set):** Contains picture encoding rules.
* `0x65` $\rightarrow$ **IDR (Keyframe / I-Frame):** A complete, full-picture image frame.
* `0x41` or `0x61` $\rightarrow$ **Non-IDR (P-Frame):** Motion delta frame.

### How Locus Uses This:
If all vendor master tables and container tags fail, Locus searches for `0x00000001 67` (SPS) and `0x00000001 65` (Keyframes). It scoops up all the raw frames, feeds them to PyAV/FFmpeg, and reconstructs playable MP4 video anyway!
