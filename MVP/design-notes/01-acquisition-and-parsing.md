# Assumptions & Decisions: Acquisition, Identification, and Parsing

*Key engineering assumptions and architectural decisions for Acquisition, Identification, and Parsing in Locus.*

---

## 1. Disk Acquisition & Ingestion Modalities
- **Initial Assumption:** Python's `open(filepath, 'rb')` alone is sufficient to protect physical evidence.
- **The Catch / Edge Case:** If an investigator mounts a raw physical drive (e.g., `/dev/sdb`), the host OS (Windows/Linux) can write hidden metadata or automount volumes, compromising forensic integrity. Standard Python `open` also crashes on physical bad sectors.
- **Final Decision:** Locus implements a dual-path ingestion architecture:
  1. **Path A: Live Physical Device Acquisition (Embedded `dc3dd`):** When connecting raw physical drives via a hardware write-blocker, Locus executes `dc3dd` as an elevated background subprocess with `conv=noerror,sync` (bad sector defense) and simultaneous dual hashing (`hash=sha256 hash=md5`) directly to a bitstream `.raw`/`.dd` image.
  2. **Path B: Pre-Existing Forensic Image Ingestion:** Direct file intake of pre-acquired `.dd`, `.raw`, or `.img` files, locked in strict read-only mode with immediate baseline dual-hash verification.

## 2. Device & OEM Identification
- **Initial Assumption:** The DVR's magic signature (e.g., `DHAV`) is always at Sector 0 or Sector 1.
- **The Catch / Edge Case:** DVRs may have standard Master Boot Records (MBR) or GUID Partition Tables (GPT) at Sector 0. The actual video data might start megabytes later. Drives can also have slight corruption at the very beginning.
- **Final Decision:** Locus will not rely on a fixed single-sector offset. It will parse standard MBR/GPT structures to find partition starts. If it's a raw unpartitioned layout, it will scan the first few megabytes (e.g., 10,000 sectors) to detect a *pattern* of signatures, rather than relying on a single absolute byte offset.

## 3. File System & Header Parsing (False Positives)
- **Initial Assumption:** Finding the bytes `DHAV` or `0x00000001` guarantees we found a video frame header.
- **The Catch / Edge Case:** Video payload data is pseudo-random. Statistically, random video bytes will occasionally match `DHAV` by pure chance. If we blindly trust it, we will extract garbage data and corrupt the stream.
- **Final Decision:** Implement **Strict Header Validation**. When `DHAV` is found, the parser must validate the subsequent bytes:
    - Is `channel_id` within a logical range (e.g., 1 to 64)?
    - Is `timestamp` a valid human date (e.g., between 2010 and 2030)?
    - Is `payload_length` a realistic size (e.g., < 5 MB)?
    If any check fails, treat the signature as a false positive and continue scanning.

## 4. The "Ring Buffer" Overwrite Problem
- **The Catch / Edge Case:** DVRs overwrite the oldest data when the disk is full. A single continuous video clip might be split: the first half sits at the very end of the physical disk, and the second half wraps around to the beginning of the disk.
- **Final Decision:** The carving engine cannot assume linear progression. The SQLite `stream_headers` map must track logical timestamps, allowing the carving engine to glue a frame from Sector 9,999,999 to the next chronological frame at Sector 2048 smoothly.
