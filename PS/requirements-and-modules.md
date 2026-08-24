# Modules & Requirements

**Back to [[locus]]**

---

## Core Modules

1. **Device & FS Identification:** Auto-detects vendor signatures, file systems, and hardware metadata for Dahua, CP Plus, Honeywell, HIKVISION, TP-Link, Godrej, Uniview, and Matrix.
2. **Bit-Stream Acquisition:** Supports forensic disk images (`.dd`, `.raw`, `.E01`) with read-only write-blocking verification.
3. **FS & Header Parsing:** Parses custom DVR filesystems (e.g., Dahua DHFS, Hikvision layouts) and unmasks raw video wrappers (`.dav`, H.264/H.265).
4. **Video Carving & Recovery:** Sector-level carving to rebuild deleted or fragmented video clips from unallocated space.
5. **Timeline Normalization:** Calibrates timestamp drift and aligns multi-camera channels onto a single synchronized timeline.
6. **Integrity & Hashing:** Computes MD5 and SHA-256 hashes at ingestion, carving, and export stages with immutable audit logs.
7. **AI & Computer Vision:** Runs YOLOv8 for object/person detection, face recognition, motion heatmaps, and event indexing.
8. **Workspace & Reporting:** Multi-camera web video player, RBAC access control, and automated court-ready PDF/HTML report export.

---

## Non-Functional Requirements

- **Legal Admissibility:** Conforms to digital evidence standards (ISO/IEC 27037) with continuous hash verification.
- **Performance:** Parallel sector carving using background worker pools.
- **Fault Tolerance:** Sector corruption in raw disk dumps logged and skipped without crashing worker tasks.
