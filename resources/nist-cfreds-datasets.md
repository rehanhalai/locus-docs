# NIST CFReDS Forensic Test Datasets & Tools

### 1. NIST File Carving Test Images
- **URL:** https://cfreds-archive.nist.gov/FileCarving/index.html
- **Purpose:** Standardized test disk images and benchmark datasets designed to validate file carving tools on fragmented and unallocated disk space.

#### Video Benchmark Images (.dd):
* **[L0_Video.dd](https://cfreds-archive.nist.gov/FileCarving/Images/L0_Video.dd.bz2)** — Non-fragmented video files (MP4, AVI, MOV, FLV, MPG, WMV). Baseline sanity check.
* **[L1_Video.dd](https://cfreds-archive.nist.gov/FileCarving/Images/L1_Video.dd.bz2)** — Sequentially fragmented video files.
* **[L2_Video.dd](https://cfreds-archive.nist.gov/FileCarving/Images/L2_Video.dd.bz2)** — Non-sequentially fragmented video files.
* **[L3_Video.dd](https://cfreds-archive.nist.gov/FileCarving/Images/L3_Video.dd.bz2)** — Video files with missing fragments (simulates overwritten sectors / circular loop buffer).
* **[L4_Video.dd](https://cfreds-archive.nist.gov/FileCarving/Images/L4_Video.dd.bz2)** — Video files nested in video files.
* **[L5_Video.dd](https://cfreds-archive.nist.gov/FileCarving/Images/L5_Video.dd.bz2)** — Braided / interleaved video files (simulates multi-channel CCTV DVR sector allocation).

---

### 2. Digital Forensic Research Test Images (DFRWS)
- **URL:** https://cfreds-archive.nist.gov/dfr-test-images.html
- **Purpose:** Public forensic test images from DFRWS challenges and NIST CFTT project for testing filesystem extraction, deleted file recovery, and memory dumps.

---

### 3. NIST String Search Test Image (Version 1.1)
- **URL:** https://cfreds.nist.gov/all/NIST/StringSearch,V11
- **Purpose:** Forensic benchmark disk image containing known target strings across various encodings (ASCII, UTF-8, UTF-16) to test keyword and pattern search algorithms.
