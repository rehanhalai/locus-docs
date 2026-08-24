# Step 4.1 Specification: MBR & Partition Table Parsing

*This document explains why we check the Master Boot Record (MBR) / Partition Tables first, the real-world problem it solves, and the step-by-step logic.*

---

## 1. What is an MBR (Master Boot Record)?

- Every hard drive is divided into **sectors** (typically 512 bytes each).
- **Sector 0** (the very first 512 bytes of the hard drive) is called the **Master Boot Record (MBR)**.
- Sector 0 has a very specific structure:
  - **Bytes 0 – 445:** Bootloader code (instructions for embedded chips).
  - **Bytes 446 – 509:** **The Partition Table** (4 entries, 16 bytes each).
  - **Bytes 510 – 511:** The MBR Magic Signature (`0x55 0xAA`).

Each 16-byte partition entry contains:
1. **Partition Type ID** (1 byte): Indicates format (e.g., `0x83` for Linux, `0x07` for NTFS, or custom vendor IDs).
2. **Starting Sector (LBA)** (4 bytes): The exact sector number where the partition begins.
3. **Total Sectors** (4 bytes): How many sectors long the partition is.

---

## 2. Why Do DVRs Have an MBR? (The Real-World Architecture)

Most modern DVRs (Dahua, Hikvision, CP PLUS) are small embedded computers running a stripped-down version of **Embedded Linux**.

When the DVR formats a new hard drive, it usually divides the disk into multiple sections (partitions):
```
[ 1TB Physical CCTV Hard Drive ]
├─► Sector 0:              MBR (Partition Table)
├─► Partition 1 (Small):   System Config / Logs (e.g., Sectors 2048 to 206847)
└─► Partition 2 (99%):     Video Storage Data (e.g., Sectors 206848 to 1,953,525,167)
                           └── This is where DHAV / HKFS video actually lives!
```

---

## 3. The Problem: What Happens If We Don't Check MBR?

- **The Naive Assumption:** Assuming the video signatures (`DHAV` / `HKFS`) start at Byte 0.
- **The Failure:** If Locus only scans Sector 0 for video headers, it will only see Linux bootloader code or MBR data. It will find **zero video headers** and falsely conclude that the hard drive is empty or unreadable!
- **The Reality:** The actual video data is sitting 1 Megabyte or several Gigabytes further down the disk at the start of Partition 2.

---

## 4. The Step-by-Step Simplified Process in Locus

When Locus opens the `.dd` image:

```
                  [ Open Sector 0 (First 512 Bytes) ]
                                   │
                                   ▼
                   [ Check Bytes 510-511 == 0x55AA? ]
                                  / \
                            YES  /   \  NO (Raw unpartitioned layout)
                                /     \
                               ▼       ▼
       [ Read 4 Partition Entries ]   [ Start scanning directly at Sector 0 ]
               │
               ▼
   [ Extract Starting Sector (LBA) ]
   (e.g., Partition 2 starts at Sector 2048)
               │
               ▼
   [ Jump file pointer to Sector 2048 ]
               │
               ▼
   [ Begin Proprietary Signature Scan (DHAV / HKFS) ]
```

### Edge Case Handled:
If a budget/legacy DVR does not use an MBR and writes raw video starting at Sector 0 ("Superfloppy" layout), Locus sees `0x55AA` is missing or invalid, and seamlessly falls back to scanning from Sector 0 without failing.
