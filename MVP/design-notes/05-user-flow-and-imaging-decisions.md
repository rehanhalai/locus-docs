# Discussion Log: User Flow, Bit-Stream Imaging, and Open-Source Strategy

*This note captures the core architectural rationale and answers key questions raised regarding user experience, bit-stream imaging, and development approach.*

---

## 1. What Happens When a User Plugs in a DVR Hard Drive? (User POV)

- **Physical Connection:** The investigator pulls the drive from the DVR, attaches it to a **Hardware Write-Blocker** (via SATA), and connects the write-blocker to the PC via USB.
- **Operating System Reaction:**
  - The OS (Windows/Linux) detects the raw disk (e.g., `PhysicalDrive1` or `/dev/sdb`), but **cannot assign a drive letter** (like `E:\`) because the filesystem is proprietary (DHAV, HKFS, etc.).
  - Windows will often trigger a dangerous pop-up: *"You must format the disk before using it"*. (User must cancel).
  - The investigator cannot browse or double-click video files in Windows Explorer.

---

## 2. Why Do We Need the Bit-Stream Backup (`.dd` / `.raw`)?

- **Evidence Preservation:** Forensic standards dictate that analysis should never be run directly on original physical media if it can be avoided. A bit-stream image is a 100% exact byte-for-byte replica of the entire drive (including deleted space, slack space, and corrupted sectors).
- **Drive Health & Longevity:** CCTV hard drives from crime scenes are often heavily worn from running 24/7. Performing repetitive carving scans directly on the physical drive can cause the drive head to permanently fail. Imaging creates one master digital copy on a fast NVMe SSD, allowing rapid, repeatable parsing without risking hardware failure.

---

## 3. Decision: Why Build Bit-Stream Acquisition Directly Into Locus?

- **User Convenience:** Rather than forcing the investigator to use 3rd-party tools (like FTK Imager or Guymager) to image the drive first and then open Locus, Locus will provide an **All-in-One Workflow**:
  ```
  Connect Drive ──► Select Drive in Locus ──► Auto Bit-Stream Clone (.dd) + Hash ──► Auto Parse & Carve
  ```
- **Alignment with Problem Statement:** SIH PS 26150 explicitly requires *"Standardized **Acquisition**, Recovery, and Analysis"*. Integrating the acquisition module fulfills the acquisition mandate directly.
- **Flexibility:** Locus also retains the option to open an already-existing `.dd` / `.raw` / `.E01` file if the user already has one.

---

## 4. Engineering Philosophy: Do We Need to Build Everything from Scratch?

**No. We do NOT need to reinvent the wheel.**

In real-world engineering, robust forensic software orchestrates proven, battle-tested open-source components:
- **Disk Imaging Engine:** Wrapped `dc3dd` / `dcfldd` (handles bad-sector zero-padding, error resilience, on-the-fly hashing).
- **Video Remuxing Engine:** `FFmpeg` / `PyAV` (packages raw H.264 elementary streams into `.mp4` containers in milliseconds without re-encoding).
- **AI Analytics Engine:** `DVR-Scan` & `Ultralytics YOLOv8` (motion detection and object identification).
- **Database Engine:** `SQLite` (high-speed indexing of sector headers and camera channels).

### What IS Our Custom Core Value?
Our custom code focuses on what open-source tools currently lack:
1. **Multi-Vendor Proprietary Filesystem Parser:** The unified engine that detects Dahua, Hikvision, CP PLUS, and generic DVR sector layouts.
2. **De-interleaving & Carving Logic:** The algorithm that reassembles fragmented, multi-camera interleaved sectors across the ring buffer into continuous video streams.
3. **Unified Forensic UI/UX:** The desktop application providing timeline sync, multi-camera playback, and PDF case reporting.
