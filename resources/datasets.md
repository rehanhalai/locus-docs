# Forensic Test Datasets & Benchmark Images

**Project:** Locus (SIH PS 26150) | **Theme:** Cybersecurity & Digital Forensics  
**Directory:** `resources/datasets.md`  
**Related Document:** [`reading-resources.md`](./reading-resources.md) *(for research papers, articles, and legal guides)*

---

## 1. NIST CFReDS File Carving Benchmark Dataset

* **Catalog URL:** https://cfreds-archive.nist.gov/FileCarving/index.html
* **Provider:** National Institute of Standards and Technology (NIST) — Computer Forensic Reference Data Sets (CFReDS).
* **Purpose:** Standardized raw bitstream disk images (`.dd`) engineered to validate and stress-test file carving algorithms against non-fragmented, fragmented, and interleaved video storage.

### Video Benchmark Images & Download Links:

| Dataset File | Direct Download Link | Fragmentation Type | Codecs / Formats Included | Purpose in Locus Testing |
| :--- | :--- | :--- | :--- | :--- |
| **`L0_Video.dd`** | [L0_Video.dd.bz2](https://cfreds-archive.nist.gov/FileCarving/Images/L0_Video.dd.bz2) | **Non-Fragmented** | MP4, AVI, MOV, FLV, MPG, WMV | Baseline sanity check for linear video frame carving. |
| **`L1_Video.dd`** | [L1_Video.dd.bz2](https://cfreds-archive.nist.gov/FileCarving/Images/L1_Video.dd.bz2) | **Sequentially Fragmented** | MP4, AVI, MOV, FLV, MPG, WMV | Tests sequential fragment reassembly when clusters are contiguous. |
| **`L2_Video.dd`** | [L2_Video.dd.bz2](https://cfreds-archive.nist.gov/FileCarving/Images/L2_Video.dd.bz2) | **Non-Sequentially Fragmented** | MP4, AVI, MOV, FLV, MPG, WMV | Tests out-of-order sector cluster reassembly using GOP/timestamp ordering. |
| **`L3_Video.dd`** | [L3_Video.dd.bz2](https://cfreds-archive.nist.gov/FileCarving/Images/L3_Video.dd.bz2) | **Missing Fragments** | MP4, AVI, MOV, FLV, MPG, WMV | Simulates sectors overwritten by circular loop buffers; tests error recovery without crashing. |
| **`L4_Video.dd`** | [L4_Video.dd.bz2](https://cfreds-archive.nist.gov/FileCarving/Images/L4_Video.dd.bz2) | **Nested Video Files** | MP4, AVI, MOV, FLV, MPG, WMV | Validates false-positive filtering when video frames exist inside other files. |
| **`L5_Video.dd`** | [L5_Video.dd.bz2](https://cfreds-archive.nist.gov/FileCarving/Images/L5_Video.dd.bz2) | **Braided / Interleaved** | MP4, AVI, MOV, FLV, MPG, WMV | **Direct simulation of multi-camera CCTV DVR sector layout** (interleaved camera channels). |

---

## 2. NIST Heimvision K9604-1 4-Channel DVR Forensic Image

* **Catalog URL:** https://cfreds.nist.gov/all/JoshBrunty,RaynaMock/HeimvisionDVRE01ForensicImage
* **Contributors:** Prof. Josh Brunty & Rayna Mock (Marshall University Digital Forensics).
* **Format:** Authentic 150 GB physical surveillance hard drive image (compressed to ~2.0 GB in Expert Witness Format `.E01`).
* **Hardware Profile:**
  - Device: Heimvision K9604-1 4-Channel NVR/DVR.
  - File System: Xiongmai / WFS 0.4 variant.
  - Recording Duration: 24-hour continuous surveillance across 4 synchronized camera feeds.
* **Included Artifacts:** Official ground-truth CSV hash lists, camera channel layout documentation, and acquisition logs.
* **Purpose in Locus:** Serves as the primary end-to-end benchmark for multi-channel video demultiplexing, WFS parsing, and timeline synchronization.

---

## 3. Digital Forensic Research Test Images (DFRWS)

* **Catalog URL:** https://cfreds-archive.nist.gov/dfr-test-images.html
* **Provider:** DFRWS Forensic Challenges & NIST CFTT Project.
* **Purpose:** Multi-scenario forensic disk images containing corrupted partition tables, FAT32/ext4/NTFS file systems, unallocated slack space, and RAM memory dumps.
* **Purpose in Locus:** Stress-testing MBR/GPT boundary parsers and unpartitioned disk fallback heuristics.

---

## 4. NIST String Search Benchmark Image (Version 1.1)

* **Catalog URL:** https://cfreds.nist.gov/all/NIST/StringSearch,V11
* **Provider:** NIST Computer Forensics Tool Testing (CFTT) Program.
* **Purpose:** Standardized test disk image containing known target strings across various encodings (ASCII, UTF-8, UTF-16LE, UTF-16BE) placed at known sector offsets.
* **Purpose in Locus:** Validates the raw sector scanning engine and regex/hex pattern matching for OEM magic byte signatures (`DHAV`, `HKFS`, `WFS`).
