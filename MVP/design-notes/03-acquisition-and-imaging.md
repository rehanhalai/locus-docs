# Acquisition & Built-In Imaging Architecture

*These notes document the physical acquisition mechanism, technical requirements, and error-handling decisions.*

---

## 1. Role of Acquisition in Locus
- **Problem Statement Alignment:** SIH 26150 explicitly calls for *"Standardized **Acquisition**, Recovery, and Analysis"*. 
- **Core Decision:** Locus will feature a **Built-in Physical Imaging & Acquisition Module**. The investigator can connect a drive via a hardware write-blocker, and Locus will handle bit-stream cloning (`.dd` creation) directly inside the application before triggering automated analysis.
- **Dual Intake Support:**
  1. **Direct Physical Disk Acquisition:** Select physical disk (`/dev/sdX` or `\\.\PhysicalDriveX`) -> Clone to `.dd` with on-the-fly hashing -> Auto-trigger parsing.
  2. **Existing Image File Ingestion:** Select pre-existing `.dd` / `.raw` / `.img` file -> Jump straight to parsing.

---

## 2. Technical Mechanisms for Disk Acquisition

### A. Raw Sector Streaming
- Locus opens the source physical block device with read-only flags (`O_RDONLY` in Linux / `GENERIC_READ` with `FILE_SHARE_READ | FILE_SHARE_WRITE` in Windows).
- Reads sequential sector blocks (recommended block size: 64 KB to 1 MB for optimal I/O throughput).
- Writes sequentially to the destination `.dd` file on the investigator's local storage.

### B. Zero-Cost On-The-Fly Hashing
- Instead of reading 1TB twice (once to copy, once to hash), Locus computes SHA-256 and MD5 hashes **simultaneously in memory** as each block is read from the physical drive.
- When copying finishes, the source hash is instantly finalized with zero additional waiting time.

---

## 3. Real-World Edge Cases & Technical Catches

### Edge Case 1: Operating System Privileges
- **The Catch:** Standard users cannot access raw block devices (`/dev/sdb` or `\\.\PhysicalDrive1`).
- **Resolution:** Locus must request elevated administrative privileges (Administrator prompt on Windows / `sudo` or raw disk permissions on Linux) when physical disk acquisition mode is selected.

### Edge Case 2: Bad Sectors & Hardware Read Errors
- **The Catch:** Surveillance hard drives endure 24/7 read/write cycles for years. Crime-scene drives frequently contain degraded or unreadable bad sectors. A standard copy command (like Python `shutil.copy` or basic read) will throw an I/O exception and crash.
- **Resolution (Forensic Zero-Padding):** When an unreadable sector is encountered:
  1. Retry the sector read (configurable, e.g., 3 retries).
  2. If still unreadable, write replacement zero-bytes (`0x00` for 512 or 4096 bytes) to the destination `.dd` file to maintain exact 1:1 physical sector offset alignment.
  3. Log the bad sector LBA (Logical Block Address) into the case audit log.

### Edge Case 3: Storage Space Verification
- **The Catch:** If the destination volume runs out of free space mid-acquisition (e.g., copying a 2TB drive onto a 500GB SSD), the process fails catastrophically midway.
- **Resolution:** Locus must perform a pre-flight disk capacity check: `free_space(destination) > total_bytes(source_drive)`. If insufficient, block acquisition before starting and prompt the user.
