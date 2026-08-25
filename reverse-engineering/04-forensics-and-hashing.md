# 4. Forensics, Hashing & Provenance Tracking

During a digital forensics investigation, evidence documentation must demonstrate byte-level integrity, transparent provenance tracking, and reproducible results.

If software outputs a media clip without proof of origin or processing history, the evidence is vulnerable to challenge for lack of provenance.

---

## 1. What is a Cryptographic Hash?
A **Hash** (`SHA-256` or `MD5`) is a deterministic mathematical algorithm that converts arbitrary file data into a fixed-length string (a digital fingerprint).

If a 500 GB file is processed through SHA-256, it generates a unique digest string (e.g., `e3b0c44298fc...`).

> **The Avalanche Effect:**  
> Changing even a single bit (0 to 1) or a single pixel in a source video file alters over 50% of the output SHA-256 hash characters, mathematically proving byte integrity.

---

## 2. Integrity, Provenance & Auditability Workflow

Locus establishes a verifiable processing chain:

1. **Baseline Ingestion Hash:** Upon ingesting a `.dd` disk image, Locus computes baseline `SHA-256` and `MD5` cryptographic hashes.
2. **Read-Only Preservation:** The source image file is opened strictly read-only; no write operations occur.
3. **Artifact Extraction Provenance:** Every derived `.mp4` video clip records its exact source disk byte offset, sector length, adapter version, and artifact `SHA-256` hash.
4. **Pre-Export Hash Verification:** Source disk image hashes are re-verified prior to report generation to confirm the original image remained unmodified throughout analysis.

In the generated **Forensic Evidence Report**, Locus details all cryptographic hashes, offset provenance tables, and processing event logs to support independent verification.
