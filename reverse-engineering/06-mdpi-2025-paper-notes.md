# Forensic Research Reference: MDPI Information (2025)

**Paper Title:** Automated Forensic Recovery Methodology for Video Evidence from Hikvision and Dahua DVR/NVR Systems  
**Authors:** L. Rzayeva, M. Shayakhmetov, Y. Atanbayev, R. Budenov, H. Mutaher  
**Journal:** *Information* (MDPI), Vol. 16, Issue 11, Article 983  
**Publication Date:** November 13, 2025  
**DOI / URL:** https://www.mdpi.com/2078-2489/16/11/983  

---

## Key Takeaways for Locus Architecture

This 2025 peer-reviewed paper specifically addresses the exact forensic problem statement that Locus is built for: automated recovery, signature matching, and timeline reconstruction from proprietary Dahua and Hikvision surveillance systems.

### 1. Dual-Signature Validation (DHFS / DHAV)
- **Header-Footer Pairing:** Instead of only scanning for the 4-byte `DHAV` header (`0x44 0x48 0x41 0x56`), the paper emphasizes verifying the trailing `dhav` footer (`0x64 0x68 0x61 0x76`) across sector boundaries.
- **Why it matters:** Ensures carved video frames are not truncated or corrupt, maintaining high court admissibility.

### 2. Adaptive Temporal Sequencing
- **Dynamic Gap Thresholds:** Rather than assuming static frame intervals, the parser dynamically calculates timestamp deltas between consecutive I-Frames / P-Frames to distinguish between:
  1. Intentional motion-detection voids (camera paused recording when scene was empty).
  2. Damaged/overwritten sectors (missing frames due to disk overwrite cycles).

### 3. Automatic Manufacturer Signature Identification
- Evaluates sector 0 / 2048 magic bytes (`DHFS`, `HIKBTREE`, `WFS0.4`) to automatically route raw bytes to the appropriate parser without investigator guesswork.
- **Reported Benchmark:** 91.8% recovery rate and 96.7% temporal accuracy across 27 tested surveillance hard drives.

---

## Relevance to Locus Implementation

| Locus Module | How it Aligns with the Paper |
| :--- | :--- |
| **`device_id`** | Matches the paper's signature-based OEM detector (Dahua DHFS vs Hikvision HKFS). |
| **`header_parser`** | Adopts the header-footer boundary validation for payload frame extraction. |
| **`video_carver`** | Stream-copies raw H.264 NAL units into standard web containers without re-encoding. |
| **`timeline_sync`** | Implements adaptive temporal delta alignment to correlate multi-camera timestamps. |
