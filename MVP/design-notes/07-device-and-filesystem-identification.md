# Step 4 Specification: Device & Proprietary Filesystem Identification

*This document outlines the exact technical process Locus executes immediately after acquiring or loading a `.dd` / `.raw` image.*

---

## 1. Why Is Identification the Mandatory Next Step?

A forensic `.dd` image is just a raw, unformatted sequence of billions of binary bytes. 
Because CCTV manufacturers do NOT use standard Windows/Linux filesystems (like NTFS, FAT32, or EXT4), standard operating systems cannot read them.

Different surveillance manufacturers use fundamentally different internal data layouts:
- **Dahua & CP PLUS:** Use the proprietary **DHAV** container format with frame-level headers (`DHAV`) and footers (`dhav`).
- **Hikvision & EZVIZ:** Use the proprietary **HKFS** (Hikvision File System) based on B+ Tree metadata indexing and data cluster blocks.
- **Swann & Generic Asian OEMs:** Use **WFS** (e.g., `WFS 0.4`) linear circular buffers.

**If Locus does not identify the filesystem first, it cannot know which parsing algorithm or sector offset map to apply.**

---

## 2. Technical Mechanism: The Multi-Vendor Signature Scanner

When the `.dd` file is opened, Locus runs a lightweight **Header Signature Scan**:

### Target Signatures & Magic Bytes:

| OEM / Vendor Family | File System Signature | Magic Bytes (Hex) | ASCII Equivalent | Structural Pattern |
| :--- | :--- | :--- | :--- | :--- |
| **Dahua / CP PLUS / Amcrest** | DHAV Container | `44 48 41 56` | `DHAV` (Header)<br>`dhav` (Footer) | Pre-pended 32/64-byte frame header wrapping H.264/H.265 |
| **Hikvision / EZVIZ** | HKFS / Hikvision FS | `48 4B 46 53` or `48 49 4B` | `HKFS` / `HIK` | B+ Tree partition tables, master metadata block |
| **Swann / Xiongmai / Generic** | WFS / ZHX | `57 46 53 20` | `WFS 0.4` | Continuous circular ring buffer without traditional files |

---

## 3. The Scanning Algorithm & Edge Case Defenses

### A. Sector-Aligned Scanning (Avoiding Slow Byte-by-Byte Scans)
- Disk controllers write in discrete sectors (512 bytes or 4096 bytes).
- Instead of scanning every single byte across 1TB (which would take hours), Locus reads **sector boundaries** (offset `0`, `512`, `1024`, `2048`, `4096`, etc.) in the first 10,000 sectors (approx. 5–40 MB).

### B. Partition Offset Traversal (Handling MBR/GPT)
- If Sector 0 contains a standard Linux/Windows MBR partition table, Locus reads the partition table entries to find the exact starting sector of the video data partition (typically Sector `2048` or `1048576` bytes in).
- Locus then inspects the start of that partition for the proprietary magic bytes.

### C. Frequency & Pattern Validation (Eliminating False Positives)
- Finding a single instance of `DHAV` could be a statistical fluke in random video data.
- **Rule:** Locus only confirms a vendor match if it detects **consistent, repeating structural headers** (e.g., finding multiple valid frame headers with matching channel IDs and incrementing timestamps).

---

## 4. UI Output to the Investigator

Once identification completes (taking under 2 seconds), Locus displays a detection summary card:
- **Detected File System:** Dahua / CP PLUS (DHAV Stream)
- **Sector Alignment:** 512-byte aligned
- **Estimated Channels:** 4 Active Camera Channels
- **Status:** Ready for Frame Indexing & Video Carving
