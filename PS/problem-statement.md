# Problem Statement: SIH 26150

**Back to [[locus]]**

---

## Background & Challenge

DVR/NVR surveillance systems across law enforcement, critical infrastructure, and commercial sites use proprietary storage formats, custom partition layouts, and non-standard video streams. Key OEM manufacturers include Dahua, CP Plus, Honeywell, HIKVISION, TP-Link, Godrej, Uniview, and Matrix.

### Key Pain Points:
1. **Vendor Fragmentation:** Custom file systems (e.g., Dahua DHFS, Hikvision raw layouts) and container headers prevent generic video playback.
2. **Tool Dependency:** Investigators must maintain multiple vendor-specific tools, risking evidence contamination and delay.
3. **Timestamp Drift:** Internal DVR clocks drift, making multi-camera event correlation difficult.
4. **Deleted Footage Recovery:** Standard file carving fails because DVR disk dumps lack standard FAT/NTFS allocation tables.
5. **Chain of Custody:** Lack of verifiable cryptographic hashes (MD5 / SHA-256) throughout extraction jeopardizes legal admissibility.

---

## Solution Objectives

Build a software platform providing:
- **Automated OEM & FS Detection:** Identify vendor signatures, file systems, and video codecs from raw disk dumps.
- **Bit-Stream Acquisition:** Create forensic images (`.dd`, `.raw`, `.E01`).
- **Parsing & Video Carving:** Parse sector headers and carve unallocated space for deleted/damaged H.264/H.265 footage.
- **Timeline Normalization:** Synchronize timestamps across heterogeneous camera feeds into a linear timeline.
- **Evidence Integrity:** Cryptographic SHA-256 and MD5 verification at all pipeline stages.
- **AI Video Analytics:** Object detection (people, vehicles), face detection, motion heatmaps, and event indexing.
- **Court-Ready Reporting:** Audit logs, hash comparison tables, and PDF evidence exports.

---

## Deliverables

1. Comparative Analysis Report of major OEM storage structures.
2. Sample DVR/NVR Forensic Images for benchmarking.
3. System Architecture Specification.
4. Functional Web & Backend Prototype.
5. Standard Operating Procedures (SOPs).
6. Validation & Cryptographic Hash Reports.
7. User Manuals & Final SIH Project Report.
