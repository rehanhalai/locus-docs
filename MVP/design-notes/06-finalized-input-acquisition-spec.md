# Finalized Specification: Input & Physical Acquisition (dc3dd Engine)

*This document represents the finalized, locked-down technical specification for Step 1 (Physical Connection) and Step 2 (Acquisition & Imaging).*

---

## 1. Input Modalities Supported

Locus supports two distinct input ingestion paths:

1. **Path A: Live Physical Device Acquisition (Primary)**
   - Target: Raw physical drive connected via hardware write-blocker (e.g., `/dev/sdX` on Linux or `\\.\PhysicalDriveX` on Windows).
   - Engine: Subprocess execution of `dc3dd`.
   - Output: Bit-stream `.raw` or `.dd` image file + verification hash log (`.log`).
   
2. **Path B: Pre-Existing Forensic Image Ingestion (Secondary)**
   - Target: An already-acquired `.dd`, `.raw`, or `.img` file stored on local storage.
   - Action: Bypasses the imaging phase and immediately enters the Device & Filesystem Identification pipeline.

---

## 2. Technical Execution of dc3dd

When physical disk acquisition is triggered from the Locus UI, Locus runs `dc3dd` as a managed background subprocess:

### Exact Command Template:
```bash
dc3dd if=<SOURCE_DEVICE> \
      of=<DESTINATION_IMAGE_PATH> \
      hash=sha256 \
      hash=md5 \
      log=<DESTINATION_LOG_PATH> \
      errlog=<DESTINATION_ERROR_LOG_PATH> \
      status=on \
      statusinterval=1 \
      conv=noerror,sync
```

### Parameter Breakdown:
* `if=<SOURCE_DEVICE>`: Source physical block device (e.g., `/dev/sdb`).
* `of=<DESTINATION_IMAGE_PATH>`: Target output path on fast local SSD (e.g., `/cases/CASE_001/evidence.raw`).
* `hash=sha256` & `hash=md5`: Enables dual on-the-fly cryptographic hashing during the byte copy.
* `log=...`: Writes complete acquisition audit trail and final hashes to a persistent text log.
* `errlog=...`: Captures exact sector numbers of any bad or degraded blocks.
* `status=on` & `statusinterval=1`: Emits real-time progress updates every 1 second to `stderr`.
* `conv=noerror,sync`: Instructs the engine not to crash on bad sectors, padding unreadable blocks with zeros (`0x00`) to preserve byte alignment.

---

## 3. Locus Subprocess & GUI Integration

1. **Process Management:** Locus executes `dc3dd` via Python's `asyncio.subprocess` (or `subprocess.Popen`).
2. **Progress Parsing:** Locus reads `stderr` line-by-line in real time, regex-parsing:
   - Bytes copied / Total bytes
   - Current throughput (MB/s)
   - Estimated Time Remaining (ETA)
3. **UI Display:** Locus updates a live progress bar, speed indicator, and sector counter in the desktop interface.
4. **Completion Handoff:**
   - Once `dc3dd` exits with return code `0`, Locus parses the generated `.log` file to extract the verified SHA-256 and MD5 hashes.
   - The resulting `.raw` / `.dd` image path is automatically passed to **Step 4: Device & Filesystem Identification**.
