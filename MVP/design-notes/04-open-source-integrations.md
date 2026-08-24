# Open-Source Tools & Integration Architecture for Locus MVP

*These notes document open-source tools and libraries that Locus can leverage under the hood for a rapid, robust MVP.*

---

## 1. Disk Acquisition & Imaging (Step 2)
Instead of writing a complex C-level disk imager from scratch, Locus can wrap battle-tested open-source forensic utilities:

- **`dc3dd` / `dcfldd` (DoD Forensics):**
  - C-based enhanced `dd` utilities designed specifically for law enforcement.
  - Built-in on-the-fly SHA-256 / MD5 hashing during copy.
  - Automatic bad-sector recovery with forensic zero-padding.
  - Standardized progress meters (`stderr`/`stdout`) that Locus can stream to display a live progress bar in the UI.
- **Pure Python Alternative (`hashlib` + block reads):**
  - A lightweight, cross-platform ~50 line Python loop reading 1MB blocks from the disk handle and updating `hashlib.sha256()` while writing to disk.

---

## 2. Proprietary Filesystem & Header Parsing (Steps 4 & 5)
Surveillance vendors use specific layouts, but open-source references exist:

- **`hikextractor` (Python):** Open-source reference parser for Hikvision DVR raw disk images, showing how to traverse Hikvision partition boundaries and sector headers.
- **`libHikvision` / `ezhikstract` (Python):** Reference implementations for parsing Hikvision/EZVIZ round-robin index files (`index00.bin`) and video data directories.
- **`dhav2mp4` / `dhav-extract`:** Open-source tools demonstrating how 32-byte Dahua `DHAV` headers wrap raw H.264 NAL units.

---

## 3. Video Remuxing & Frame Assembly (Step 6)
- **`FFmpeg` / `PyAV` (Python bindings for FFmpeg C libraries):**
  - Industry-standard tool to take extracted raw H.264/H.265 Elementary Streams (`.h264`) and repackage them into `.mp4` containers.
  - Remuxes in milliseconds without re-encoding (preserving exact bit integrity).

---

## 4. Video Analytics & Motion Detection (Step 8)
- **`DVR-Scan` (Python):** Open-source command-line tool specifically designed to analyze security footage and detect motion events, extracting only relevant activity.
- **`Ultralytics YOLOv8` (Python):** Lightweight open-source model for local AI object detection (Persons, Vehicles, License Plates).
