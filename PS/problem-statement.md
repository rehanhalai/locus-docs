# Detailed Problem Analysis & Strategic Objectives: SIH 26150

**Back to [[locus]]**

---

## 1. What Problem Locus Solves

Locus addresses the critical problem of **surveillance video evidence fragmentation, proprietary disk layout opacity, timestamp distortion, and unvalidated recovery** during law enforcement and digital forensics investigations.

When physical surveillance hard drives or raw disk images are seized from DVR/NVR units manufactured by OEMs such as **Dahua, Hikvision, CP Plus, Honeywell, TP-Link, Godrej, Uniview, and Matrix**, digital forensic examiners face extreme technical barriers in extracting, reconstructing, synchronizing, and reporting video evidence.

---

## 2. Why the Problem Exists

Surveillance DVR/NVR hardware is designed for low-cost, continuous multi-channel real-time video recording on mechanical drives. To optimize write performance and minimize drive wear:
- **Bypassing Standard Filesystems:** Devices frequently bypass standard operating system filesystems (NTFS, FAT32, EXT4).
- **Raw Ring-Buffer Allocation:** Disk space is formatted as massive pre-allocated raw ring buffers divided into proprietary sector blocks.
- **Custom Frame Multiplexing:** Video frames from 4, 8, 16, or 32 camera channels are multiplexed and interleaved into raw disk blocks with custom proprietary headers.
- **Index Overwriting:** Master index tables are updated periodically in fixed disk locations. When drive capacity is reached, older index entries are overwritten while video frames remain in raw sector blocks until physically written over.

---

## 3. Why the Problem is Technically Difficult

1. **Heterogeneous Proprietary Formats:** Different vendors (and even different firmware versions from the same vendor) use distinct sector block headers, magic byte signatures, and container structures.
2. **Missing or Damaged Index Tables:** Power loss, abrupt power cuts during incidents, or deliberate drive formatting destroys or corrupts the master index table, leaving raw video frames orphaned across terabytes of disk space.
3. **Non-Standard Time Encodings:** In-stream timestamps are often stored in custom binary formats, Unix epoch variants, or proprietary BCD (Binary Coded Decimal) fields, skewed by internal DVR clock drift.
4. **Interleaved Multi-Channel Streams:** Consecutive disk sectors contain interleaved frames from different cameras. Naive file carvers merge frames from multiple cameras into a single scrambled, unplayable video file.
5. **Lack of Standard Tooling:** Commercial forensic tools are often locked to standard operating system filesystems and fail to correctly interpret raw proprietary surveillance layouts.

---

## 4. Why Incorrect Recovery is Dangerous

> [!CAUTION]
> **Forensic Risk Warning:**  
> **A wrong-but-plausible video stream is far more dangerous than an explicit extraction failure.**

In a criminal investigation or legal proceeding, incorrect recovery leads to catastrophic consequences:
- **Frame Misordering & Stream Corruption:** Merging non-sequential video frames creates artificial motion sequences that misrepresent suspect actions.
- **Channel Cross-Contamination:** Attributing video footage from Camera 2 (e.g., an alleyway) to Camera 1 (e.g., a cash register) creates false investigative leads.
- **Distorted Timestamps:** Failing to account for clock drift or misinterpreting timezone offsets corrupts suspect alibis and event timelines.
- **Unvalidated Carving Output:** Presenting a carved video clip without verified source offsets and checksums renders the evidence vulnerable to challenge for lack of provenance.

---

## 5. What Happens If This Is Not Solved?

- **Critical Evidence Lost:** Surviving deleted or expired video footage remains hidden in unallocated sectors.
- **Investigative Delays:** Examiners lose days attempting to manually parse raw disk sectors using generic hex editors.
- **Evidence Invalidation:** Unverified video conversions without hash parity or processing history tracking fail independent scrutiny.
- **Blind Reliance on Unchecked AI:** Using AI detection on unvalidated or corrupted video frames creates false positives and misidentifications.

---

## 6. Solution Objectives

Locus establishes a forensically sound engineering platform built around **read-only image analysis, modular vendor adapters, media validation, timestamp normalization, transparent provenance, and secondary AI triage**:

1. **Pre-Acquired Image Ingestion & Dual Hashing:** Read-only ingestion of `.dd`, `.raw`, and `.img` forensic images with concurrent streaming `SHA-256` and `MD5` baseline verification.
2. **Device & Storage Layout Scanner:** Automated sector-aligned magic header scanner to identify vendor layout signatures without forcing inaccurate classifications.
3. **Modular Vendor Adapters:** Validated adapters (`DahuaDHAVAdapter`, `HikvisionHKFSAdapter`) for index-aware recovery and metadata-guided sector carving.
4. **Stream & Media Validation Engine:** Container demuxing and frame continuity checks assigning explicit recovery statuses (`RECOVERED`, `PARTIAL`, `CORRUPTED`, `UNRECOVERABLE`).
5. **Raw & UTC Normalized Timeline:** Preservation of original in-stream timestamps alongside normalized master UTC timeline calculation for multi-camera correlation.
6. **Cryptographic Provenance & Audit Logging:** Traceability of every derived clip to exact source disk byte offsets, sector lengths, adapter versions, and hash parity records.
7. **Secondary AI Triage Layer:** Local CPU inference via ONNX Runtime for object/motion candidate search with mandatory Human-in-the-Loop review.
8. **Forensic Evidence Reporting:** Structured PDF/HTML reports detailing case evidence, baseline hashes, artifact provenance, and explicit uncertainty disclaimers.
