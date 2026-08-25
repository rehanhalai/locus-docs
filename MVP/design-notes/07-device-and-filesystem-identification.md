# Step 4 Specification: Device & Proprietary Filesystem Identification

**Back to [[MVP/MVP|MVP]]**

---

## 1. Why Is Identification the Mandatory Next Step?

A forensic `.dd` image is a raw sequence of binary bytes. 
Because surveillance DVR manufacturers do not utilize standard operating system filesystems (such as NTFS, FAT32, or EXT4), standard operating systems cannot mount or interpret them.

Different surveillance manufacturers use fundamentally different internal data layouts:
- **Dahua Technology:** Uses proprietary **DHAV** container formats and DHFS indexing tables.
- **CP Plus:** Uses custom OEM profiles (often DHAV-compatible, requiring profile validation via `CPPlusAdapter`).
- **Hikvision & EZVIZ:** Use the proprietary **HKFS** (Hikvision File System) based on B+ Tree metadata indexing and data cluster blocks.
- **Swann & Generic OEMs:** Use **WFS** (e.g., `WFS 0.4`) linear circular buffers.

**If Locus does not identify the storage layout first, it cannot select the correct parsing adapter or sector mapping algorithm.**

---

## 2. Technical Mechanism: The Multi-Vendor Signature Scanner

When the `.dd` file is opened, Locus executes a **Header Signature Scan**:

### Candidate Signatures & Magic Bytes:

| OEM / Vendor Family | File System Signature | Magic Bytes (Hex) | ASCII Equivalent | Structural Pattern | Validation Status |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Dahua Technology** | DHAV Container | `44 48 41 56` | `DHAV` (Header)<br>`dhav` (Footer) | Pre-pended frame header wrapping H.264/H.265 bitstreams | Partially Validated |
| **CP Plus** | DHAV Variant / Custom | `44 48 41 56` / Custom | `DHAV` / OEM Tag | Custom frame wrappers evaluated via `CPPlusAdapter` | Researching *(Requires lab data)* |
| **Hikvision / EZVIZ** | HKFS / Hikvision FS | `48 4B 46 53` or `48 49 4B` | `HKFS` / `HIK` | B+ Tree partition tables, master metadata block | Partially Validated |
| **Swann / Generic** | WFS / ZHX | `57 46 53 20` | `WFS 0.4` | Continuous circular ring buffer without traditional files | Researching |

---

## 3. The Scanning Algorithm & Edge Case Defenses

### A. Sector-Aligned Scanning
- Storage controllers read and write in discrete sectors (512 bytes or 4096 bytes).
- Locus reads **sector boundaries** across the initial partition space to identify candidate headers efficiently without full-disk byte-by-byte scanning.

### B. Partition Offset Traversal (Handling MBR/GPT)
- If Sector 0 contains a standard MBR partition table, Locus parses the partition table entries to locate the starting sector of the video data partition (typically Sector `2048` / 1 MB offset).
- Locus then inspects the start of that partition for master superblock or frame signatures.

### C. Frequency & Pattern Validation (Eliminating False Positives)
- Finding a single instance of `DHAV` could be a statistical coincidence in random video payload data.
- **Rule:** Locus only confirms a vendor match if it detects **consistent, repeating structural headers** (e.g., matching channel IDs, incrementing timestamps, and valid payload lengths).

---

## 4. UI Output to the Investigator

Once identification completes, Locus displays a detection summary card:
- **Candidate Layout:** Dahua Technology (DHAV Stream Profile)
- **Selected Adapter:** `DahuaDHAVAdapter` (Partially Validated)
- **Sector Alignment:** 512-byte aligned
- **Estimated Channels:** 4 Active Camera Channels
- **Confidence Status:** `LIKELY` (Ready for Master Sector Mapping)
