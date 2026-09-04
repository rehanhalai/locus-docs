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

### 1. MDPI Information (2025 Paper)
* **Title:** *"Automated Forensic Recovery Methodology for Video Evidence from Hikvision and Dahua DVR/NVR Systems"*
* **Journal:** MDPI *Information*, 16(11), 983 (Published 2025)
* **DOI / URL:** [10.3390/info16110983](https://www.mdpi.com/2078-2489/16/11/983)
* **Authors:** L. Rzayeva, M. Shayakhmetov, Y. Atanbayev, R. Budenov, H. Mutaher.
* **Information Used in Locus:**
  * Automated manufacturer identification using magic bytes (`DHAV`, `HKFS`).
  * Adaptive temporal sequencing algorithms to reconstruct fragmented video streams.
  * Dual-signature header-footer validation (`DHAV`...`dhav`) yielding a 91.8% recovery rate across 27 physical surveillance hard drives.

### 2. MDPI Journal of Forensic Sciences
* **Title:** *"Forensic Video Recovery from Multi-Channel Analog DVR Systems"*
* **Information Used in Locus:**
  * Exact 32-byte `DHAV` frame header structure: magic `0x44484156`, 1-byte channel ID, 1-byte frame type (`0xFD` I-Frame, `0xFC` P-Frame), 4-byte payload size, bit-packed timestamp, and `0x64686176` (`dhav`) footer.
  * De-interleaving interleaved multi-camera sector streams into distinct channel queues.

### 3. Digital Forensics Magazine
* **Title:** *"Forensic Analysis of the Hikvision File System"*
* **Information Used in Locus:**
  * Hikvision HKFS Master Sector headers (LBA 0 / LBA 2048), sector size (512 bytes), data block cluster size (2 MB / 16 MB).
  * Hierarchical `HIKBTREE` index layout mapping recording time ranges to cluster addresses.
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
