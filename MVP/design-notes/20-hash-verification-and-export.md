# Step 9 Specification: Hash Verification & Integrity Export

*This document outlines the cryptographic and operational protocols used by Locus to guarantee that exported video evidence maintains a mathematically provable chain of custody, ensuring forensic integrity and verifiable provenance.*

---

## 1. The Evidence Lineage & Integrity Challenge

When an investigator extracts a 5-minute `.mp4` clip from a 1-Terabyte surveillance drive, they are essentially creating a *new* file. A defense attorney will naturally challenge the authenticity of this new file:
- *"How do we know the police didn't alter the footage?"*
- *"How do we know this isn't an AI-generated deepfake?"*
- *"Where is the proof that this exact file came from the seized DVR?"*

**The Goal of Step 9:** To create a mathematically unbreakable link between the original seized hard drive (the physical evidence) and the exported `.mp4` file (the digital presentation).

---

## 2. The Zero-Transcoding Guarantee

To maintain true forensic integrity, Locus enforces a strict **Zero-Transcoding Policy** during export.
- **No Re-encoding:** The system does not decode and re-encode the video pixels (which causes generation loss and degrades image quality).
- **Raw Byte Slicing:** Locus uses FFmpeg stream copying (`-c:v copy`) to lift the exact H.264/H.265 NAL units directly from the physical disk sectors and place them into an `.mp4` container.
- **Result:** The video stream data is bit-for-bit identical to the video data written by the DVR motherboard.

---

## 3. The Cryptographic Hash Chain

When the investigator finalizes their trimmed clip and clicks "Export," Locus executes a streamlined provenance sequence:

1. **Read-Only Verification:** Locus operates under strict read-only file locks (`O_RDONLY`), referencing the verified baseline SHA-256 hash generated during Step 1 (Ingestion).
2. **Export Video:** The raw NAL units are sliced from the exact disk sector offsets and remuxed into `suspect_clip.mp4` (zero transcoding).
3. **Generate Output Artifact Hash:** Locus immediately calculates the cryptographic SHA-256 and MD5 hashes of the newly created `suspect_clip.mp4`.
4. **On-Demand Whole-Disk Verification:** Full re-verification of the multi-terabyte `.dd` file is performed asynchronously on-demand (`POST /api/verify`) or during case closure, avoiding multi-hour I/O bottlenecks during individual clip exports.

---

## 4. The Digital Audit Sidecar (`.sync.json`)

To prove the lineage of the new file, Locus generates a cryptographic "Receipt" that lives alongside the exported video. This is typically a `.json` file (and an accompanying human-readable `.pdf`).

### Example Sidecar Schema:
```json
{
  "export_metadata": {
    "investigator_id": "Officer Smith (Badge: 1042)",
    "export_timestamp_utc": "2026-08-25T14:30:00Z",
    "case_number": "CR-2026-8942",
    "software_version": "Locus Forensics v1.0.0"
  },
  "source_evidence": {
    "source_type": "Physical Drive (/dev/sdb)",
    "source_master_sha256": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"
  },
  "extraction_parameters": {
    "channel_id": 2,
    "start_sector": 1450204,
    "end_sector": 1461900,
    "applied_time_offset_ms": 300000,
    "calibration_method": "Manual OSD Sync"
  },
  "output_evidence": {
    "filename": "suspect_clip.mp4",
    "output_sha256": "9f86d081884c7d659a2feaa0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a08"
  }
}
```

---

## Appendix: Plain English Terminology

If the technical flow above seems abstract, here is how you can explain **Hash Verification and the Audit Trail** to a non-technical judge, jury, or project manager:

### The "Bloody Knife" (Physical Chain of Custody)
Imagine a detective finds a bloody knife at a crime scene. They put it in an evidence bag and sign their name across the tape seal. They hand it to the crime lab, and the lab tech signs the seal. They bring it to court, and the judge looks at all the unbroken seals and signatures. Because the seals are unbroken, the court knows with 100% certainty that nobody swapped the knife for a fake one.

### The "ATM Receipt" (Digital Chain of Custody)
In our case, the "bloody knife" is the 1-Terabyte DVR hard drive seized from the shop. But in court, we aren't showing the jury all 1-Terabyte of data—we are showing them a 5-minute video clip. The defense lawyer will argue that we faked the video.

To prove the 5-minute video is real, Locus prints a hyper-secure **Digital Receipt** (the Audit Sidecar) that travels everywhere with the video. 

This receipt states:
> *"I started with the original hard drive (and I verified its fingerprint). I went exactly to Sector X and copied the raw data. I did **not** change a single pixel. I put those bytes into a new file. The fingerprint for this new file is XYZ. Officer Smith authorized this extraction."*

When the defense lawyer challenges the video, the investigator simply tells the court to run their own fingerprint (hash) on the video file. It will equal `XYZ`, proving mathematically that the video has not been tampered with since the exact moment Locus extracted it.
