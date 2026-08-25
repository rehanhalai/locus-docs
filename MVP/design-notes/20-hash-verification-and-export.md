# Step 9 Specification: Cryptographic Hash Verification & Provenance Export

**Back to [[MVP/MVP|MVP]]**

---

## 1. Provenance Verification & Derived Artifact Distinction

When an investigator extracts a video clip from a multi-terabyte surveillance disk image, the resulting `.mp4` file is a **DERIVED ARTIFACT**.

In forensic examinations, establishing authenticity requires documenting:
- The exact source evidence image from which bytes were carved.
- The precise byte offset and sector range.
- The extraction method and parser version used.
- The cryptographic hash of both the source evidence and the newly created derived artifact.

> [!IMPORTANT]
> **Source Evidence Hash vs. Derived Artifact Hash:**  
> The master evidence is the pre-acquired forensic image file (`.dd`, `.raw`, `.img`). An exported `.mp4` video file contains new container metadata (MP4 moov/mdat atoms) wrapping the extracted bitstream. Therefore:  
> $$\text{Original Evidence Hash} \neq \text{Derived MP4 Hash}$$  
> Locus independently computes and records both hashes in SQLite and the `.sync.json` provenance sidecar.

---

## 2. Stream-Preserving Remuxing (No Re-Encoding)

To preserve forensic integrity during clip export, Locus utilizes **stream-preserving remuxing**:
- **No Transcoding:** The system does not decode and re-compress video frames, preventing generational pixel degradation and preserving the original compressed elementary stream.
- **Stream-Copy Packaging:** Locus uses PyAV/FFmpeg in stream-copy mode (`-c:v copy`) to package raw H.264/H.265 NAL units directly into standard `.mp4` containers.
- **Forensic Preservation:** The compressed video payload bytes in the derived MP4 match the source disk bitstream.

---

## 3. Cryptographic Verification Sequence

During clip export, Locus executes a sequential integrity routine:

1. **Verify Source Image Integrity:** Locus computes a current SHA-256 hash across the source `.dd` image and compares it to the baseline hash registered during Step 3 (Ingestion). A match confirms the evidence image remained unmodified.
2. **Stream Extraction & Remux:** The designated byte range is carved and remuxed into `derived_clip.mp4`.
3. **Generate Artifact Digest:** Locus immediately computes `SHA-256` and `MD5` cryptographic digests of the derived `.mp4` artifact.
4. **Generate Provenance Sidecar:** Writes the `.sync.json` descriptor file recording extraction parameters, parent evidence hashes, and artifact digests.

---

## 4. Digital Provenance Sidecar Schema (`.sync.json`)

```json
{
  "export_metadata": {
    "investigator_id": "Officer Smith (ID: 1042)",
    "export_timestamp_utc": "2026-08-25T14:30:00Z",
    "case_number": "CASE-2026-8942",
    "software_version": "Locus Forensics v1.0.0"
  },
  "source_evidence": {
    "source_type": "Forensic Disk Image (.dd)",
    "source_file_name": "dahua_nvr_dump.dd",
    "source_master_sha256": "8c2a5f4b9d1e38a7c6e0f2b4d6a8c1e3f5b7d9a0c2e4f6a8b1d3f5a7c9e1b3d5",
    "source_master_md5": "c4f8a1e2d3b5c7a9e0f1b3d5a7c9e1b3",
    "pre_export_status": "VERIFIED_MATCH"
  },
  "extraction_parameters": {
    "channel_id": 2,
    "source_start_byte_offset": 104857600,
    "source_length_bytes": 14857600,
    "applied_time_offset_ms": 300000,
    "calibration_method": "OSD_VISUAL_ANCHOR",
    "adapter_used": "DahuaDHAVAdapter v1.2.0",
    "transcoding": "NONE_STREAM_COPY_ONLY"
  },
  "derived_artifact": {
    "filename": "derived_clip_ch02.mp4",
    "artifact_sha256": "9f86d081884c7d659a2feaa0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a08",
    "artifact_md5": "a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6"
  }
}
```

---

## 5. Summary of Forensic Value

1. **Transparent Lineage:** Every derived clip links mathematically to its source image byte range.
2. **Defensible Verification:** Independent examiners can re-verify both the master image hash and the derived artifact hash using standard cryptographic CLI utilities (`sha256sum`).
3. **Audit Trail Completeness:** Processing parameters, investigator identity, and adapter versions are recorded immutably.
