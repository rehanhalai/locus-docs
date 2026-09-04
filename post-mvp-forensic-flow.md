# Locus: Post-MVP Standard Forensic Lifecycle & Enterprise Roadmap

**Project:** Locus (SIH PS 26150) | **Theme:** Cybersecurity & Digital Forensics  
**Document Purpose:** Defines the complete, gold-standard, production-grade surveillance forensic lifecycle for Indian Cyber Cells, State FSLs (Maharashtra, Gujarat, Telangana, Karnataka), and CFSL Hyderabad under the **Bharatiya Sakshya Adhiniyam (BSA) 2023** and **Bharatiya Nagarik Suraksha Sanhita (BNSS) 2023**.

> [!TIP]
> **Hackathon Strategy & Pitch Value:**  
> Use this document to demonstrate **deep domain mastery to SIH judges**.  
> The live hackathon MVP demonstrates the high-performance core engine (dc3dd acquisition, proprietary filesystem parsing, zero-transcoding carving, 60 Hz master timeline, and local ONNX AI).  
> This roadmap documents the **complete courtroom-ready forensic lifecycle** that proves Locus is designed as a field-deployable national law enforcement suite, not just an academic prototype.

---

## 1. The Complete 12-Stage Standard Forensic Flow (The Golden Standard)

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                        THE 12-STAGE PRODUCTION FORENSIC FLOW                           │
├────────────────────────────────────────────────────────────────────────────────────────┤
│ PHASE 1: ON-SCENE SEIZURE & PROVENANCE                                                 │
│  [01. Scene Photo Vault] ──► [02. Master Clock Drift Anchor] ──► [03. Custodian Memo]  │
│                                                                           │            │
│ PHASE 2: HARDWARE ACQUISITION & INTEGRITY                                 ▼            │
│  [06. RAID Reassembly] ◄── [05. Retention Estimator] ◄── [04. dc3dd Write-Blocker]     │
│             │                                                                          │
│             ▼                                                                          │
│ PHASE 3: DEEP CARVING & METADATA                                                       │
│  [07. Device & Cloud ID] ──► [08. Sector & Log Parsing] ──► [09. Zero-Transcode Carve]│
│                                                                           │            │
│ PHASE 4: INTELLIGENCE & COURTROOM ADMISSIBILITY                           ▼            │
│  [12. Courtroom Pack & BSA 63] ◄── [11. Local ONNX AI] ◄── [10. Calibrated IST Sync]   │
└────────────────────────────────────────────────────────────────────────────────────────┘
```

---

### Phase 1: Physical Scene Seizure & Visual Chain-of-Custody

#### Stage 01: On-Scene Photographic Evidence Vault
* **The Forensic Reality:** Defense advocates routinely challenge digital evidence by claiming: *"The DVR was physically damaged, turned off, or tampered with before police arrived."*
* **Post-MVP Implementation:**
  - Case Intake provides a structured **Photo Upload Vault** with mandatory photo categories:
    1. **Front Panel & Status LEDs:** Captures Make, Model, Serial, and the **blinking RECORDING LED** (on-scene proof that the recorder was actively writing video at seizure).
    2. **Rear Panel & Cabling:** Captures active BNC coax inputs, Ethernet PoE ports, and power supply.
    3. **Internal Chassis & Storage:** Captures HDD serial labels, manufacturer dates, and SATA channel port mappings.
  - Locus immediately generates an immutable **SHA-256 cryptographic hash** for each photo, logging them into the tamper-proof case audit ledger.

#### Stage 02: Master Clock Drift Anchor (The "Golden Photo")
* **The Forensic Reality:** Surveillance DVR clocks almost never synchronize with NTP servers. They operate on cheap quartz RTC chips with dying backup batteries, drifting **1 to 5 minutes per month**.
* **Post-MVP Implementation:**
  - Mandatory capture of the **Drift Anchor Photograph**: The live DVR monitor showing the on-screen clock photographed side-by-side with an NTP-synchronized wristwatch/smartphone in the same camera frame.
  - Locus records the exact offset ($\Delta t_{\text{seizure}} = t_{\text{DVR}} - t_{\text{NTP}}$) and locks the baseline for downstream linear drift backward calculation.

#### Stage 03: Proprietor / Custodian Intake Questionnaire (BSA Section 63 Memo)
* **The Forensic Reality:** Under **BSA 2023 Section 63** (formerly IEA Section 65B), electronic records are inadmissible without an affirmative statement from the person responsible for the system (*Anvar P.V. v P.K. Basheer*).
* **Post-MVP Implementation:**
  - Integrated digital **Custodian Panchnama Form**:
    - Custodian Identity: Full Name, Phone, Government ID (Aadhaar/Voter ID), Official Role (*e.g., "Proprietor, Ambika Jewellers, Surat"*).
    - Installation Context: Purpose of surveillance, physical location, ordinary course of business.
    - Statutory Declarations (Radio Toggles):
      - ☑ *"The surveillance system was functioning properly without malfunction."*
      - ☑ *"The recordings were generated in the ordinary course of business."*
      - ☑ *"The recorder was under my lawful custody without unauthorized modification."*
    - Signature Pad for on-scene digital sign-off, or instant generation of a printable 1-page physical seizure memo for ink signature.

---

### Phase 2: Hardware Acquisition & Infrastructure Defense

#### Stage 04: Hardware Write-Blocker Ingestion (`dc3dd` Pipeline)
* **Live MVP Feature:** Integrated elevated execution of `dc3dd` with `conv=noerror,sync` (bad-sector defense) and simultaneous dual on-the-fly hashing (`hash=sha256 hash=md5`).
* **Post-MVP Enhancement:** Automated hardware write-blocker detection via USB/SATA bridge descriptor querying (verifying that Tableau, WiebeTech, or CRU write-blockers are actively write-protecting the bus).

#### Stage 05: Pre-Acquisition Ring Buffer Retention Feasibility Estimator
* **The Forensic Reality:** DVRs write into fixed-size circular ring buffers. Once full, old footage is permanently overwritten. Imaging a 4 TB drive takes hours; investigators need to know immediately if the footage could even have survived.
* **Post-MVP Implementation:**
  - Mathematical Feasibility Calculator:
    $$\text{Retention (Days)} \approx \frac{\text{Drive Usable Capacity (GB)}}{\text{Active Cameras} \times \text{Nominal Bitrate (Mbps)} \times 10.8}$$
  - If a 4 TB drive with 16 cameras at 4 Mbps wraps in ~5.8 days, and the crime was 12 days ago, Locus warns:  
    *⚠️ "High Overwrite Probability: Local ring buffer has wrapped ~2.1 times since the incident date. Physical sectors likely overwritten."*

#### Stage 06: Multi-Disk RAID Reassembly (RAID 0, 1, 5, 10 & JBOD)
* **The Forensic Reality:** Commercial NVRs (e.g. Hikvision DS-76xxNI, Dahua NVR5xxx) use 2 to 8 hard drives in RAID 0, RAID 1, or RAID 5 configurations. Slicing individual disks in isolation produces fragmented half-streams.
* **Post-MVP Implementation:**
  - Ingestion of multiple bitstream images (`evidence_disk1.raw`, `evidence_disk2.raw`).
  - Parsing RAID superblock metadata (chunk size, stripe order, parity rotation) to virtually assemble a unified block device without altering source images.

---

### Phase 3: Deep Carving & Telemetry Correlation

#### Stage 07: Device Identification & Cloud NVR Defense
* **The Forensic Reality:** Post-2020 installations (Hik-Connect, CP Plus iCloud, EZVIZ) store only a **sparse local cache** (motion clips or last 24h) on-premise; the continuous master video resides in vendor cloud data centers.
* **Post-MVP Implementation:**
  - Automated detection of Cloud-Recording NVR firmware profiles.
  - Generates an automated statutory **BNSS Section 94 Production Notice** addressed to the vendor's Indian Nodal Officer (e.g., Prama Hikvision India Pvt Ltd) requesting the cloud return.
  - Ingestion channel for **Cloud Evidence Returns** (vendor MP4 exports + XML manifests) to display side-by-side with local disk fragments.

#### Stage 08: Master Sector Mapping & DVR System Event Log Carving
* **The Forensic Reality (*Tomaso Bruno* Presumption):** Supreme Court precedent (*Tomaso Bruno v State of UP, 2015*) establishes that unexplained CCTV timeline gaps trigger an adverse inference against the prosecution under BSA Section 119 (evidence suppression).
* **Post-MVP Implementation:**
  - Deep sector carving decodes both video stream headers and **DVR System Event Log sectors** (`system_event_logs` table).
  - Logs sensor states, motion trigger start/stop events, power outages, and administrative logins.

#### Stage 09: GOP-Aligned Video Carving & Zero-Transcoding Remuxing
* **Live MVP Feature:** Sector-level carving snapping cleanly to nearest I-Frames, wrapping raw NAL units into standard `.mp4` containers via PyAV/FFmpeg stream copying (`-c copy`) to preserve original codec hashes.

---

### Phase 4: Multi-Angle Intelligence & Courtroom Production

#### Stage 10: Linear Clock Drift Propagation & Dual-Playhead Master Timeline
* **Post-MVP Implementation:**
  - Propagates quartz RTC drift linearly backwards to the target incident window:
    $$\text{Calibrated IST Wall Time}(t) = t_{\text{DVR}} - \left[\Delta t_{\text{seizure}} - (\text{Drift Rate} \times \text{Days Prior})\right]$$
  - Playhead features a **Dual-Clock Display**:
    - `[Raw OSD Time: 21:42:35]` (pixel fidelity for courtroom cross-examination).
    - `[Calibrated IST: 21:45:35]` (matches Call Detail Records, mobile tower dumps, and police General Diary entries).
  - Gaps on the timeline are affirmatively stamped with green verification badges:  
    `[ SENSOR IDLE — NO MOTION DETECTED (VERIFIED BY SYSTEM LOG) ]`

#### Stage 11: Local Edge AI Intelligence (ONNX YOLOv8 + Forensic Heatmaps)
* **Live MVP Feature:** OpenCV MOG2 motion void detection + local YOLOv8 ONNX Runtime inference (persons, vehicles) with sub-second SQLite parameterized queries.
* **Post-MVP Enhancement:** Cross-camera re-identification (ReID) embeddings allowing investigators to click a suspect in Camera 1 and automatically highlight matching appearances across Cameras 2, 3, and 4 on the timeline.

#### Stage 12: Statutory Court Evidence Production Pack (BSA 2023 Section 63)
* **Post-MVP Implementation:**
  - 1-Click compilation of the **Complete Court Evidence Package**:
    1. **Statutory BSA 2023 Section 63 Admissibility Certificate:** Formally pre-filled with hardware serials, physical SATA port mapping, dual cryptographic hashes (SHA-256 + MD5), and signed custodian/examiner declarations.
    2. **Annexure A (Photo Provenance):** High-resolution on-scene photos of the DVR front panel, blinking recording LED, HDD serials, and the side-by-side clock photo.
    3. **Annexure B (Numbered I-Frame Stills):** Uncompressed PNG stills (`Ex-CCTV-001.png`) of critical moments, each stamped with individual SHA-256 hashes for paper charge sheet submission.
    4. **Annexure C (Clock Drift Calibration Table):** Mathematical derivation proving how DVR OSD times map to Indian Standard Time.
    5. **Annexure D (System Log Motion Integrity Statement):** Affirmative proof satisfying BSA Section 119 that timeline gaps were natural sensor voids, not deleted evidence.
    6. **Cryptographic Proof Sidecars:** `.sync.json` files verifying byte-for-byte provenance back to the raw disk bitstream image.

---

## 2. MVP vs. Post-MVP Feature Matrix (The Pitch Boundary)

Use this table during hackathon evaluations to clearly delineate what is working in the live demo versus what is engineered in the enterprise roadmap:

| Capability Area | Live Hackathon MVP (Working Demo) | Post-MVP Enterprise Roadmap (Vision & Pitch) |
| :--- | :--- | :--- |
| **Disk Acquisition** | Embedded `dc3dd` live disk cloning + pre-existing `.dd` ingestion with dual SHA-256/MD5 hashing | Automated hardware write-blocker detection; multi-drive RAID 0/1/5 virtual reassembly |
| **Scene Provenance** | Case metadata intake (FIR, IO, Station) | Interactive Custodian Declaration Form + On-Scene Photo Evidence Vault (blinking LED, clock photo) |
| **Filesystem Parsing** | 2-tier magic signature scanning (`DHAV`, `HKFS`, `WFS`) across 8 OEMs | Dedicated System Event Log carving (`system_event_logs`) to refute *Tomaso Bruno* suppression claims |
| **Cloud Surveillance** | Detects local filesystem layout | Cloud NVR Profile detection + automated BNSS Section 94 Production Notice generator for cloud Nodal Officers |
| **Video Carving** | GOP-snapped zero-transcode remuxing (`-c copy`) via PyAV/FFmpeg | Damaged sector interpolation + chip-off NAND binary carving for physically burnt/waterlogged DVR boards |
| **Timeline Sync** | 60 Hz master clock bus + relative multi-camera synchronization | On-Scene Seizure NTP Clock Drift Calibration + Dual Timestamp playhead (Raw OSD vs. Calibrated IST) |
| **AI Analytics** | Local YOLOv8 ONNX object detection (persons, vehicles) + OpenCV MOG2 motion void detection | Cross-camera suspect re-identification (ReID) + Automated License Plate Recognition (ALPR) |
| **Court Reporting** | Audit trail ledger export + `.sync.json` cryptographic sidecars | Full Statutory **BSA 2023 Section 63 Certificate Generator** + 1-Click Numbered I-Frame Stills Exhibit Pack |

---

## 3. Hackathon Judge Pitching Guide & Q&A Defenses

When pitching to senior police officers, forensic scientists, or SIH jury panels, use these structured defenses:

### Q1: "CCTV clocks are always wrong in small shops. How does your timeline hold up in court?"
> *"Judges, that is the single biggest point of failure in surveillance forensics. In real casework, defense advocates destroy CCTV timelines by comparing them against Call Detail Records (CDR) and mobile tower dumps. Locus solves this through our **Seizure Clock Drift Protocol**: the officer photographs the DVR clock next to an NTP wristwatch at seizure. Locus anchors this offset and linearly propagates quartz drift backwards to the exact second of the crime, displaying dual timestamps—Raw OSD for pixel fidelity and Calibrated IST for tower correlation."*

### Q2: "What if there are gaps in the recording? How do you prove police didn't delete evidence?"
> *"Under the Supreme Court's landmark ruling in **Tomaso Bruno v State of UP (2015)** and **BSA 2023 Section 119**, unexplained CCTV gaps create an adverse presumption against the prosecution. Most small-shop DVRs record on motion detection. Locus doesn't just carve video; it parses the **DVR's internal System Event Log partition**. When a gap appears, Locus proves from the machine's own logs that the sensor was idle and no motion occurred, affirmatively protecting the investigation against tampering claims."*

### Q3: "Newer CCTV systems upload footage to the cloud. How does your tool handle that?"
> *"In post-2020 installations like Hik-Connect and CP Plus iCloud, the local hard drive is merely a sparse temporary cache. Imaging the local disk in isolation yields incomplete evidence. Locus detects cloud-recording profiles, alerts the examiner, and auto-generates a statutory **BNSS Section 94 Notice** to the vendor's Indian Nodal Officer. It then allows ingesting the vendor's cloud return alongside local physical disk fragments on a unified timeline."*

### Q4: "Is your output admissible under the new Indian criminal laws?"
> *"Yes. Under **Bharatiya Sakshya Adhiniyam (BSA) 2023 Section 63**, electronic evidence requires strict statutory certification from the lawful custodian. Locus automates this entire process: it generates the formal Section 63 certificate pre-filled with device serial numbers, dual SHA-256/MD5 hashes, custodian declarations, and extracts uncompressed **I-Frame keyframe stills** (`.png`) with individual cryptographic hashes as physical court annexures."*
