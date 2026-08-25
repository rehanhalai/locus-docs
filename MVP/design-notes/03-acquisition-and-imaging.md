# Acquisition & Image Ingestion Architecture

**Back to [[MVP/MVP|MVP]]**

---

## 1. Role of Image Ingestion in Locus

- **Primary Input Boundary:** Pre-acquired forensic disk image ingestion (`.dd`, `.raw`, `.img`).
- **Software Read-Only Protection:** Ingested disk image files are opened using strict software read-only handles (`rb` mode in Python / `O_RDONLY` at OS level) to prevent application-level modification.
- **Physical Drive Acquisition:** Upstream physical drive cloning (via hardware write-blockers) is categorized as a Phase 3 roadmap item.

---

## 2. Technical Mechanisms for Image Ingestion

### A. Raw Sector Streaming
- Locus opens the source image file in read-only mode.
- Reads sequential sector blocks using 64 KB streaming buffers for efficient I/O.
- Calculates baseline `SHA-256` and `MD5` cryptographic hashes concurrently during block reading without requiring a separate post-hashing pass.

---

## 3. Real-World Edge Cases & Technical Controls

### Edge Case 1: Operating System File Permissions
- Read-only file handle access is verified prior to opening evidence files.

### Edge Case 2: Bad Sector Null-Padding
- If a pre-acquired disk image contains unreadable or zero-filled sector ranges, Locus logs the sector offsets in `audit_log` and flags derived media clips from those sectors as `CORRUPTED` or `PARTIAL`.

### Edge Case 3: Storage Capacity Pre-Flight Check
- Pre-flight workspace disk space check: `free_space(derived_dir) > expected_derived_storage`.
