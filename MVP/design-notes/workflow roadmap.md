1. **Step 1: Physical Device Seizure & Connection** (Handling the physical DVR/HDD, hardware write-blockers).
2. **Step 2: Forensic Imaging & Acquisition** (Creating the bit-stream copy, e.g., `.dd` or `.E01`).
3. **Step 3: Software Ingestion & Baseline Hashing** (Loading the image into Locus and securing the cryptographic baseline).
4. **Step 4: Device & Filesystem Identification** (Detecting the DVR OEM and proprietary structure).
5. **Step 5: Header Parsing & Sector Mapping** (Locating and decoding video frame metadata).
6. **Step 6: Video Carving, Assembly & Remuxing** (Extracting raw frames and wrapping them into playable containers).
7. **Step 7: Timeline Normalization & Sync** (Aligning timestamps across multiple camera channels).
8. **Step 8: AI-Assisted Analytics** (Running local object detection).
9. **Step 9: Hash Verification & Chain of Custody Audit** (Ensuring evidence integrity prior to export).
10. **Step 10: Forensic Report Generation** (Producing the court-admissible PDF).