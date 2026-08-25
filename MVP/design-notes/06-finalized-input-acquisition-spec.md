# Specification: Input Ingestion & Upstream Acquisition (dc3dd Architecture)

**Back to [[MVP/MVP|MVP]]**

---

## 1. Input Modalities & MVP Scope Boundary

Locus strictly defines its input ingestion boundary to maintain forensic defensibility:

### Path A: Pre-Acquired Forensic Image Ingestion (Primary MVP Scope)
- **Target:** An already-acquired bit-stream disk image (`.dd`, `.raw`, `.img`) stored on local storage.
- **Access Mode:** Ingested strictly via software read-only handles (`'rb'` in Python / `O_RDONLY` at OS level).
- **Action:** Calculates baseline cryptographic hashes (`SHA-256`, `MD5`) via streaming block reads and directly enters the Device & Filesystem Identification pipeline.

### Path B: Live Physical Device Acquisition (Future Capability / Outside Current MVP Scope)
- **Target:** Raw physical mechanical/solid-state drive connected via a dedicated hardware write-blocker (e.g., `/dev/sdX` on Linux or `\\.\PhysicalDriveX` on Windows).
- **Engine:** Managed background subprocess execution of `dc3dd` or `dcfldd`.
- **Status:** **Outside MVP Scope / Phase 3 Roadmap Item**. Upstream acquisition is assumed to be performed prior to loading evidence images into Locus.

> [!IMPORTANT]
> **Hardware Write Blocker vs. Application Read-Only Mode:**  
> A hardware write-blocker prevents electrical write signals at the hardware/controller layer when imaging physical media. Opening a `.dd` file read-only in software prevents application-level write operations to an already-created image file. **These are not equivalent.** Locus MVP operates post-acquisition on disk image representations.

---

## 2. Technical Architecture for Future Physical Acquisition (`dc3dd`)

For future roadmap integration (Phase 3), physical disk acquisition will wrap `dc3dd` as a managed subprocess:

### Subprocess Command Template:
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
* `if=<SOURCE_DEVICE>`: Source physical block device (e.g., `/dev/sdb` or `\\.\PhysicalDrive1`).
* `of=<DESTINATION_IMAGE_PATH>`: Target raw bit-stream output path on local storage (e.g., `/cases/CASE_001/evidence.raw`).
* `hash=sha256` & `hash=md5`: Enables dual on-the-fly cryptographic hashing during byte streaming.
* `log=...`: Writes complete acquisition audit log and baseline hashes to a persistent log file.
* `errlog=...`: Records exact sector numbers of unreadable or degraded blocks.
* `status=on` & `statusinterval=1`: Streams periodic progress metrics to `stderr`.
* `conv=noerror,sync`: Prevents process termination on bad sectors, padding unreadable blocks with zeros (`0x00`) to preserve sector alignment.

---

## 3. Subprocess Management & UI Integration (Future Implementation)

1. **Process Orchestration:** Locus will manage the imager via Python's `asyncio.subprocess`.
2. **Progress Telemetry:** Stream `stderr` line-by-line, extracting:
   - Bytes copied / Total capacity
   - Instantaneous throughput (MB/s)
   - Estimated Time Remaining (ETA)
3. **Completion Handoff:**
   - On exit code `0`, Locus extracts verified SHA-256 and MD5 digests from the `.log` file.
   - The verified `.raw` / `.dd` image path is then handed off to **Feature 02: Device & Filesystem Identification**.
