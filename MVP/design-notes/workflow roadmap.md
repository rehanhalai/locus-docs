# Forensic Investigation Workflow Roadmap

**Back to [[MVP/MVP|MVP]]**

---

### Phase A: Upstream Pre-Ingestion Workflow (Hardware / External Imager)
1. **Step 1: Physical Device Seizure & Hardware Write-Blocking** (Handling physical DVR/HDD with dedicated hardware write-blocker; upstream process).
2. **Step 2: Upstream Forensic Imaging** (Generating bit-stream disk image, e.g., `.dd`, `.raw`, `.img`).

---

### Phase B: Locus MVP Forensic Analysis & Processing Engine
3. **Step 3: Forensic Image Ingestion & Baseline Hashing** (Loading `.dd` image in read-only mode, computing baseline SHA-256/MD5).
4. **Step 4: Device & Filesystem Layout Identification** (Multi-tier signature and partition table inspection).
5. **Step 5: Sector-Level Metadata & Header Parsing** (Decoding frame headers, populating SQLite Master Sector Map).
6. **Step 6: Video Stream Carving, Assembly & Remuxing** (Extracting raw bitstreams, GOP keyframe alignment, zero-transcoding remuxing into derived MP4 artifacts).
7. **Step 7: Timeline Preservation & Master UTC Normalization** (Preserving raw timestamps, applying non-destructive clock drift and timezone calibration).
8. **Step 8: Local Secondary AI Analytics (ONNX Triage)** (Local YOLOv8 candidate object detection with Human-in-the-Loop verification).
9. **Step 9: Cryptographic Hash Parity & Provenance Sidecar Export** (Re-verifying source image hashes, hashing derived artifacts, exporting `.sync.json`).
10. **Step 10: Forensic Evidence Report Generation** (Compiling audit trails, hash tables, and provenance chains into HTML/PDF evidence reports).