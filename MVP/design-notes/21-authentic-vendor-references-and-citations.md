# 21. Authentic Vendor References, Forensic Papers & Benchmark Datasets

*This document compiles all verified academic research papers, patents, open-source reverse-engineering repositories, and public forensic benchmark datasets for CCTV/DVR file systems.*

---

## 1. Dahua Technology & CP PLUS (DHFS 4.1 / DHAV)

Dahua and CP PLUS utilize the proprietary **DHFS** (Dahua File System) and wrap individual frames in **DHAV** 32-byte binary envelopes.

### Published Research & Reverse-Engineering Sources:
1. **Academic Paper (2025):** *"Automated Forensic Recovery Methodology for Video Evidence from Hikvision and Dahua DVR/NVR Systems"* (MDPI *Information*, 16(11), 983). [DOI: 10.3390/info16110983](https://www.mdpi.com/2078-2489/16/11/983)
   - Authors: L. Rzayeva, M. Shayakhmetov, Y. Atanbayev, R. Budenov, H. Mutaher.
   - Core concepts: Adaptive temporal sequencing, dual-signature header-footer validation (`DHAV`...`dhav`), automated manufacturer identification (91.8% recovery rate on 27 surveillance drives).
2. **Academic Paper:** *"Forensic Video Recovery from Multi-Channel Analog DVR Systems"* (MDPI Journal of Forensic Sciences / Digital Investigation).
   - Details the 32-byte `DHAV` frame header structure: magic `0x44484156`, 1-byte channel ID, 1-byte frame type (`0xFD` I-Frame, `0xFC` P-Frame), 4-byte payload size, bit-packed timestamp, and `0x64686176` (`dhav`) footer.
2. **X-Ways Forensics X-Tension:** [dw2102/DHFS4_1](https://github.com/dw2102/DHFS4_1)
   - Open-source implementation parsing Dahua DHFS 4.1 sector allocation blocks and frame-level demultiplexing.
3. **Open-Source Stream Decoders:**
   - [cyraxjoe/dav2mp4](https://github.com/cyraxjoe/dav2mp4)
   - [PeterTheobald/Dav2Mp4](https://github.com/PeterTheobald/Dav2Mp4)

---

## 2. Hikvision & EZVIZ (HKFS / HIKBTREE)

Hikvision DVRs utilize the **HKFS** (Hikvision File System) and an indexed master **B+ Tree** architecture (`HIKBTREE`).

### Published Research & Reverse-Engineering Sources:
1. **Academic Paper:** *"Analysis of the HIKVISION DVR File System"* by Jaehyeok Han, Doowon Jeong, Sangjin Lee (Korea University), published in *Digital Forensics and Cyber Crime* (Springer LNICST Vol. 157, pp. 189–199, DOI: `10.1007/978-3-319-25512-5_13`, ResearchGate: 285429692).
   - Documents the Master Sector header (`HIKVISION@HANGZHOU` at offset `0x200`), sector size (512 bytes), data block cluster size (1 GB / 2MB / 16MB), System Logs partition, and the hierarchical `HIKBTREE` index layout storing channel recording ranges.
2. **Forensic Tools & Extraction Repositories:**
   - [vishwajitsarnobat/HIKVISION-DVR-Tool](https://github.com/vishwajitsarnobat/HIKVISION-DVR-Tool): Full parsing of HKFS filesystem, HIKBTREE parsing, and stream remuxing.
   - [fmpfeifer/hikextractor](https://github.com/fmpfeifer/hikextractor): Python extractor for Hikvision raw disk partitions.
3. **Patent Reference:** CN101895874A (Hikvision Video Storage and Indexing System).

---

## 3. Swann, Xiongmai & Asian White-Label DVRs (WFS 0.4)

WFS (WFS 0.4) is a widely deployed circular buffer file system across thousands of generic and OEM DVR devices.

### Official Forensic Datasets & Tools:
1. **NIST CFReDS Benchmark Image:**
   - [NIST Heimvision K9604-1 4-Channel DVR Forensic Image](https://cfreds.nist.gov/all/JoshBrunty,RaynaMock/HeimvisionDVRE01ForensicImage) (Contributed by Prof. Josh Brunty & Rayna Mock, Marshall University).
   - Authentic 150 GB (2.0 GB compressed `.E01`) 4-camera 24-hour continuous surveillance recording with official ground-truth CSV hash lists.
2. **Reverse Engineering Tools:**
   - `WFS-DVR-Carver` & `wfs-extractor`: Documenting the 4-byte `WFS\x00` / `WFS 0.4` superblock descriptor at LBA 0 and the 32MB/64MB circular channel blocks.

---

## 4. International Standards (Partitioning & Video Codecs)

1. **MBR Partition Table Standard:**
   - IEEE / BIOS Standard: 512-byte Sector 0 (LBA 0), `0x55AA` signature at offset 510–511, 4x 16-byte partition entries starting at offset 446 (`0x01BE`).
2. **GPT Partition Table Standard:**
   - UEFI Specification: LBA 1 header signature `0x45 0x46 0x49 0x20 0x50 0x41 0x52 0x54` (`EFI PART`).
3. **ITU-T H.264 / ISO/IEC 14496-10 (AVC) Standard:**
   - Byte stream start codes (`0x00000001`) with NAL unit types: SPS (`0x67`), PPS (`0x68`), IDR Keyframe (`0x65`).

---

## 5. Public Forensic Reference Repositories

- **NIST Computer Forensic Reference Data Sets (CFReDS):** https://cfreds.nist.gov/
- **Digital Corpora Forensic Disk Images:** https://digitalcorpora.org/corpora/disk-images/
- **Digital Forensic Research Workshop (DFRWS):** https://dfrws.org/
