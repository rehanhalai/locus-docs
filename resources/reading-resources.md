# Reading Resources, Research Papers & Forensic Guides

**Project:** Locus (SIH PS 26150) | **Theme:** Cybersecurity & Digital Forensics  
**Directory:** `resources/reading-resources.md`  
**Related Document:** [`datasets.md`](./datasets.md) *(for disk images and test datasets)*

---

## 1. Digital Forensics Field Guides & Casework (India)

### ForensicSpot — DVR/NVR and Surveillance System Forensics (June 2026)
* **URL:** https://forensicspot.com/topics/digital-forensics/dvr-nvr-and-surveillance-system-forensics
* **Primary Scope:** Comprehensive operational guide on how Indian cyber cells, State FSLs (Maharashtra, Gujarat, Telangana, Karnataka), and CFSL Hyderabad acquire and parse CCTV evidence.
* **Core Topics & Information Used in Locus:**
  * **Recorder Architectures:** DVR (analog coax BNC), NVR (digital PoE Ethernet), and Cloud-Recording NVR (Hik-Connect, CP Plus iCloud, EZVIZ).
  * **Storage Architecture:** Fixed 256 MB segment circular ring buffers; data destruction mechanics when the ring wraps; slack space recovery.
  * **Codecs & Containers:** H.264 (AVC) and H.265 (HEVC) GOP structures; standalone I-Frames vs predictive P/B frames; Dahua `.dav` container encapsulation; zero-transcoding re-muxing (`-c copy`) for cryptographic preservation.
  * **The Clock Drift Problem:** Standalone DVR RTC clocks drifting 1 to 5 minutes/month; on-scene wristwatch vs. DVR screen photo calibration; linear backward propagation formula to correlate timestamps with Call Detail Records (CDR) and mobile tower dumps.
  * **The *Tomaso Bruno* Presumption Defense:** Distinguishing motion-triggered gaps from deleted footage using internal DVR system event logs to satisfy BSA Section 119.
  * **Legal Admissibility:** Bharatiya Sakshya Adhiniyam (BSA) 2023 Section 63 statutory certificate requirements and BNSS Section 94 production notices for cloud NVR providers.

#### Related Indian Forensic Curriculum Modules on ForensicSpot:
* **BNS 2023 Cyber Provisions and BSA 2023 Electronic Evidence:**  
  https://forensicspot.com/topics/digital-forensics/bns-2023-cyber-and-bsa-2023-electronic-evidence
* **The Digital First Responder (Volatility, Seizure & Imaging):**  
  https://forensicspot.com/topics/digital-forensics/digital-first-responder-volatility-seizure-imaging
* **Data Recovery & File Carving (Deleted, Hidden & Encrypted Content):**  
  https://forensicspot.com/topics/digital-forensics/data-recovery-file-carving-deleted-encrypted-content
* **Cloud Forensics & Multi-Tenant Challenges:**  
  https://forensicspot.com/topics/digital-forensics/cloud-forensics-multi-tenant-api-jurisdictional-challenges

---

## 2. Academic Research Papers & Reverse-Engineering Studies

### 1. MDPI Information & ResearchGate (2025 Benchmark Paper)
* **Title:** *"Automated Forensic Recovery Methodology for Video Evidence from Hikvision and Dahua DVR/NVR Systems"*
* **Authors:** Leila Rzayeva, M. Shayakhmetov, Yernat Atanbayev, Ruslan Budenov, Hamza Mutaher Alshameri (CyberTech Research Center, Astana IT University).
* **Journal:** MDPI *Information*, 16(11), 983 (Published November 13, 2025).
* **DOI:** [10.3390/info16110983](https://doi.org/10.3390/info16110983)
* **ResearchGate Publication:** [ResearchGate: 397624940](https://www.researchgate.net/publication/397624940_Automated_Forensic_Recovery_Methodology_for_Video_Evidence_from_Hikvision_and_Dahua_DVRNVR_Systems)

#### Paper Overview & Abstract:
Digital video surveillance systems are ubiquitous in modern security infrastructure, but proprietary file systems deployed by major manufacturers (Dahua, Hikvision) represent a severe bottleneck in digital forensic investigations. This paper proposes an automated forensic recovery methodology for Hikvision and Dahua systems utilizing direct binary analysis, frame-based parsing, and automated video reconstruction.

#### The Three Core Methodological Innovations:
1. **Adaptive Temporal Sequencing:**  
   Dynamically adjusts gap-detection thresholds based on frame arrival deltas rather than assuming fixed frame rates. This enables the parser to reliably distinguish between intentional motion-detection voids (sensor idle) and damaged/overwritten sectors (loss due to circular ring buffer overwrite).
2. **Dual-Signature Header–Footer Validation:**  
   Instead of relying solely on leading 4-byte headers (`DHAV` / `0x44 0x48 0x41 0x56`), the algorithm verifies the trailing `dhav` footer (`0x64 0x68 0x61 0x76`) across sector boundaries. This eliminates false-positive signature hits within pseudo-random video payload data and guarantees carved frames are structurally complete.
3. **Automatic Manufacturer Identification:**  
   Performs sector-aligned scanning across LBA 0 / 2048 to identify partition layouts and OEM magic bytes (`DHFS`, `HIKBTREE`, `WFS0.4`), automatically routing disk sectors to the correct decoding engine without investigator guesswork.

#### Empirical Benchmark Results (Tested on 27 Surveillance Drives):
* **91.8% Overall Recovery Rate:** Successfully reconstructed valid video streams across physical test hard drives.
* **96.7% Temporal Accuracy:** Accurate chronological ordering of carved events.
* **2.4% False Positive Rate:** Lowest among all tested forensic tools, demonstrating statistically significant improvement over commercial suites ($p < 0.01$).
* **87.2% Recovery on Fragmented Streams:** Outperformed leading commercial forensic tools (which averaged 82.4%) on non-sequential fragmented sectors.
* **Court Admissibility & Algorithmic Transparency:** Delivers verifiable mathematical reproducibility without proprietary black-box obscurity, with zero-transcoding conversion to `.mp4` keeping original codec metadata intact.

### 2. Springer LNICST & ResearchGate (Seminal Hikvision HKFS Paper)
* **Title:** *"Analysis of the HIKVISION DVR File System"*
* **Authors:** Jaehyeok Han, Doowon Jeong, Sangjin Lee (Center for Information Security Technologies, Korea University).
* **Conference/Publisher:** *Digital Forensics and Cyber Crime* (7th EAI International Conference, ICDF2C 2015) / Springer Lecture Notes of the Institute for Computer Sciences, Social Informatics and Telecommunications Engineering (LNICST), Vol. 157, pp. 189–199.
* **DOI:** [10.1007/978-3-319-25512-5_13](https://doi.org/10.1007/978-3-319-25512-5_13)
* **ResearchGate Publication:** [ResearchGate: 285429692](https://www.researchgate.net/publication/285429692_Analysis_of_the_HIKVISION_DVR_file_system)

#### Paper Overview & Abstract:
HIKVISION Digital Video Recorders (DVRs) dominate the global surveillance market but utilize a proprietary storage architecture that standard operating systems (Windows, Linux, macOS) cannot mount or interpret, prompting operating systems to report the drive as "Unallocated" or "Not Initialized" and falsely prompting users to format the disk. This seminal reverse-engineering study is the first systematic academic analysis of the physical disk layout, partition geometry, and metadata indexing architecture of the HIKVISION file system.

#### The Four Core File System Sections Reverse-Engineered:
1. **Master Sector (Offset `0x200` / LBA 1):**
   * **Magic Signature:** `HIKVISION@HANGZHOU` at physical offset `0x200` (512-byte boundary).
   * **Volume Geometries:** Contains total disk capacity, allocation block size, total block counts, and disk initialization/format timestamps.
   * **Partition Pointers:** Explicit 64-bit LBA offsets and sector lengths pointing to:
     * System Logs Area
     * Video Data Area
     * Primary `HIKBTREE` Index Area
     * Redundant Backup Master Sector (located adjacent to system logs for disaster recovery).
2. **`HIKBTREE` (Hierarchical B+ Tree Index):**
   * **Index Mechanism:** Maintains a multi-level B+ Tree indexing every recording segment across all connected camera channels.
   * **Node Metadata:** Each index node stores Channel ID (Camera #), Recording Start Timestamp (UTC), Recording End Timestamp (UTC), Physical Cluster/Block Offset, Recording Trigger Mode (Continuous/Normal, Motion Detection, Alarm), and Block Allocation Status.
   * **Forensic Utility:** Eliminates the need for 12+ hour exhaustive brute-force carving; Locus can traverse the B+ tree in $< 3$ seconds to reconstruct the complete multi-camera video directory with millisecond timestamps.
3. **Video Data Area (Large Pre-Allocated Blocks):**
   * Pre-allocates massive data blocks (typically ~1 GB per block, subdivided into 2 MB / 16 MB clusters) to prevent disk fragmentation and head thrashing during concurrent 24/7 multi-channel recording.
   * Houses raw video elementary payloads (H.264 / H.265 / private Hikvision encapsulation) sequentially written per channel.
4. **System Event Logs Area:**
   * Contains hardware and operational logs: system boot/shutdown, user logins, configuration modifications, recording start/stop events, camera video signal losses, and manual disk formatting triggers.

#### Forensic Recovery & Anti-Forensic Implications:
* **Absence of File-Level Deletion:** Hikvision DVRs do not implement individual file deletion. Storage operates as a circular FIFO ring buffer overwriting the oldest data blocks first.
* **Orphaned Index Recovery:** When footage is overwritten or when a suspect executes a "Format Disk" command via the DVR GUI, residual B+ tree leaf nodes and system event log entries frequently remain intact in unallocated metadata sectors. This enables forensic examiners to prove what video previously existed, the exact time ranges recorded, and whether gaps were caused by natural FIFO overwrites or deliberate anti-forensic formatting (crucial for satisfying BSA 2023 Section 119 and rebutting *Tomaso Bruno* adverse inferences).

### 3. Complementary Hikvision System Log Forensic Study (2023)
* **Title:** *"IoT forensics: Exploiting unexplored log records from the HIKVISION file system"*
* **Authors:** Evangelos Dragonas et al.
* **Journal:** *Journal of Forensic Sciences*, August 2023.
* **DOI:** [10.1111/1556-4029.15349](https://doi.org/10.1111/1556-4029.15349)
* **Information Used in Locus:** Deep decoding of proprietary binary system log records (event codes, power state transitions, and user audit trails) stored in the Hikvision log partition.

### 4. MDPI Journal of Forensic Sciences (Multi-Channel Analog Extraction)
* **Title:** *"Forensic Video Recovery from Multi-Channel Analog DVR Systems"*
* **Information Used in Locus:**
  * Exact 32-byte `DHAV` frame header structure: magic `0x44484156`, 1-byte channel ID, 1-byte frame type (`0xFD` I-Frame, `0xFC` P-Frame), 4-byte payload size, bit-packed timestamp, and `0x64686176` (`dhav`) footer.
  * De-interleaving interleaved multi-camera sector streams into distinct channel queues.
* **Patent Reference:** CN101895874A (*Hikvision Video Storage and Indexing System*).

---

## 3. Indian Statutory Authorities & Judicial Precedents

* **Bharatiya Sakshya Adhiniyam (BSA) 2023:**
  * **Section 63:** Admissibility of electronic records (mandatory statutory certificate signed by lawful custodian and examiner).
  * **Section 119:** Adverse presumption against party withholding evidence (governs missing CCTV timeline gaps).
* **Bharatiya Nagarik Suraksha Sanhita (BNSS) 2023:**
  * **Section 94:** Production notice for documents or things (used to compel cloud NVR evidence from Indian Nodal Officers).
  * **Section 175:** Notice and procedure during police inquiry and investigation.
* **Supreme Court of India Precedents:**
  * ***Anvar P.V. v P.K. Basheer (2014) 10 SCC 473:*** Electronic evidence is strictly inadmissible without statutory certificate compliance.
  * ***Tomaso Bruno v State of UP (2015) 7 SCC 178:*** Withholding available CCTV footage or failing to explain timeline gaps creates an adverse inference against the prosecution.
