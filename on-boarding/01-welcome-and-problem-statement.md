# 01 — Welcome & The Problem Statement

Welcome to the **Locus** team! This document explains the real-world problem we are solving, why traditional tools fail, and the legal principles guiding our platform.

---

## 1. The Real-World Crime Scene Scenario

Imagine a serious incident occurs at a commercial complex monitored by 16 CCTV cameras. Police arrive, seize the CCTV recorder (DVR/NVR) from the security room, extract the 1-Terabyte hard drive, and attempt to view the footage on a police workstation.

When they plug that drive into a standard computer running Windows, macOS, or standard Linux desktop, they hit a wall:

```
[ Seized CCTV Hard Drive ] ──► Plugged into Windows/Mac ──► 💥 "Disk is not formatted. Format now?"
```

---

## 2. Why Can't Normal Computers Read CCTV Hard Drives?

1. **Proprietary Filesystems (No Windows/Mac compatibility):**
   - CCTV DVR manufacturers (Dahua, Hikvision, CP Plus, Uniview, Honeywell) do **not** format hard drives with standard filesystems like NTFS, FAT32, or EXT4.
   - To maximize continuous 24/7 video write speeds and reduce manufacturing costs, they design **custom, proprietary disk formats** (e.g., `DHFS`, `HKFS`, `WFS`).

2. **Interleaved (Scrambled) Video Streams:**
   - Because 16 cameras record simultaneously around the clock, the DVR dumps fragments of video from Camera 1, Camera 7, and Camera 3 in an intermixed, sequential stream across physical disk sectors.
   - Without custom decoding, the data looks like random digital noise.

3. **The 480-Day Review Bottleneck:**
   - 16 cameras recording continuously for 30 days generates roughly **480 camera-days of video**.
   - An investigator looking for a 10-second burglary is forced to manually scrub and fast-forward through weeks of empty hallways and dark parking lots.

---

## 3. What is Locus?

**Locus** is a unified, vendor-agnostic desktop forensic analysis platform for surveillance evidence. It takes raw, unreadable CCTV hard drives from any major manufacturer and delivers:
- Bit-stream forensic disk cloning with bad-sector protection.
- Automated brand & filesystem identification.
- Proprietary sector header parsing and zero-transcoding video carving into standard `.mp4`.
- Multi-camera timeline synchronization with non-destructive calibration.
- Local, offline AI analytics (motion void filtering + YOLOv8 object detection).
- Fast parameterized evidence search (Google-like search bar for CCTV).
- Cryptographic chain-of-custody verification and legal PDF reporting.

```
┌─────────────────────────┐      ┌──────────────────────────┐      ┌─────────────────────────────┐
│ Unreadable CCTV Drive   │ ──►  │    LOCUS FORENSIC SUITE  │ ──►  │ 🔍 Searchable Multi-Cam     │
│ (Dahua, Hikvision, etc.)│      │ (Decodes, Syncs, AI Scan)│      │ ⚖️ Court-Certified Evidence │
└─────────────────────────┘      └──────────────────────────┘      └─────────────────────────────┘
```

---

## 4. The 3 Golden Rules of Forensics

Every engineer, designer, and researcher on this team must understand these 3 legal rules:

### Rule 1: Never Touch the Original Evidence (Strict Read-Only)
- In a court of law, if software alters even one byte of metadata or updates a file timestamp on the seized drive, defense attorneys can argue evidence tampering, and the entire case will be thrown out.
- Locus always operates through hardware write-blockers and strict read-only file handles (`open(path, 'rb')`).

### Rule 2: Mathematics is the Legal Witness (Cryptographic Hashing)
- Every image and exported video clip receives a unique mathematical fingerprint (**SHA-256** and **MD5**).
- Due to the *Avalanche Effect*, modifying a single pixel changes more than 50% of the hash characters. Hashes prove beyond mathematical doubt that our digital copy is 100% faithful to the original.

### Rule 3: Never Re-Encode (Zero-Transcoding Stream Copy)
- We never decode and re-compress video pixels (which causes quality loss and changes pixel values).
- We extract the exact compressed video frames (H.264/H.265 NAL units) bit-for-bit and remux them into a standard `.mp4` container.

---

*Next up: Read [02-core-features-and-analogies.md](./02-core-features-and-analogies.md) to see how the 8 features work using simple real-world analogies.*
