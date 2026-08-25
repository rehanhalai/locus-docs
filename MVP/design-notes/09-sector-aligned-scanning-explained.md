# Step 4.2 Specification: Sector Boundary Scanning

*This document explains what sector boundary scanning is, why it is essential for performance, and how it works mathematically.*

---

## 1. The Physical Reality: How Hard Drives Write Data

Hard drives and SSDs do not write individual random bytes. Physical storage controllers read and write data in fixed hardware chunks called **Sectors**:
- **Standard Sector Size:** 512 bytes
- **Advanced Format (4Kn) Sector Size:** 4,096 bytes (4 KB)

Because of this hardware convention, DVR firmware **typically starts writing a new frame header at the beginning of a sector boundary**. In all validated vendor profiles to date, headers are sector-aligned; however, uncharacterized firmware revisions or budget OEMs may deviate from this convention.

Therefore, headers are expected at mathematical multiples of 512:
- **Sector 0:** Byte `0`
- **Sector 1:** Byte `512`
- **Sector 2:** Byte `1024`
- **Sector 3:** Byte `1536`
- **Sector N:** Byte `N * 512`

---

## 2. The Problem: Byte-by-Byte vs. Sector-Aligned Scanning

### Approach A: The Slow Naive Way (Byte-by-Byte)
In a 1-Terabyte `.dd` forensic image, there are **1,000,000,000,000 bytes** (1 trillion bytes).
If a script inspects every single byte:
> *"Is byte 0 DHAV? Is byte 1 DHAV? Is byte 2 DHAV? ... Is byte 999,999,999,999 DHAV?"*

- **Result:** 1 Trillion loop iterations. The CPU runs at 100% load, the scan takes hours or days, and the software becomes unresponsive.

### Approach B: The Forensic Way (Sector-Aligned Scanning)
Because we know the physical sector rule, we **skip 511 bytes of payload data** after every check:
> *"Check byte 0 $\rightarrow$ Skip 511 bytes $\rightarrow$ Check byte 512 $\rightarrow$ Skip 511 bytes $\rightarrow$ Check byte 1024..."*

- **Result:** The scanner performs **512 times fewer operations**. A scan across the first 10,000 sectors (approx. 5 Megabytes) completes in **less than 50 milliseconds**.

---

## 3. What Locus Does at Each Sector Boundary

At each 512-byte interval, Locus reads the first 4 bytes and compares them against known vendor signatures:

```python
# Conceptual Python Sector-Boundary Scanner
SECTOR_SIZE = 512
offset = starting_sector * SECTOR_SIZE

with open("evidence.raw", "rb") as f:
    f.seek(offset)
    for sector_idx in range(10000): # Sample first 10,000 sectors
        magic = f.read(4) # Read first 4 bytes of sector
        
        if magic == b"DHAV":
            # Potential Dahua / CP PLUS detected
        elif magic == b"HKFS":
            # Potential Hikvision detected
        elif magic == b"WFS ":
            # Potential Swann / Generic detected
            
        f.seek(SECTOR_SIZE - 4, os.SEEK_CUR) # Fast-forward to the next sector start
```

---

## 4. Hardware Edge Case: 512-byte vs. 4096-byte (4K) Alignment
- Modern large-capacity drives (e.g., 4TB+ Western Digital Purple surveillance drives) often use 4,096-byte physical sectors.
- Since $4096 = 512 \times 8$, a 512-byte sector scanner will **automatically hit every 4,096-byte boundary** as well. This provides robust coverage across both legacy 512-byte drives and modern 4K Advanced Format drives.
