# Discussion Log: User Flow, Bit-Stream Imaging, and Open-Source Strategy

**Back to [[MVP/MVP|MVP]]**

---

## 1. Physical Device Connection vs Image Ingestion

- **Physical Connection (Upstream Process):** The investigator extracts the storage drive from the seized DVR/NVR, attaches it to a **Hardware Write-Blocker** (via SATA/SAS), and connects the write-blocker to the workstation.
- **Operating System Reaction:**
  - Standard host operating systems (Windows/macOS) detect the physical drive block device, but **cannot assign a native volume** because the filesystem is proprietary (DHFS, HKFS, etc.).
  - Windows may prompt: *"You must format the disk before using it"*. (The investigator must cancel this dialog).
  - The investigator uses standard forensic imaging tools (e.g., `dd`, `dc3dd`, FTK Imager) with hardware write-blocking to create a bit-stream disk image (`.dd`, `.raw`, `.img`).

---

## 2. Role of Bit-Stream Images (`.dd` / `.raw`)

- **Evidence Preservation:** Forensic standards require that primary analysis operate on exact forensic representations rather than working directly on physical evidence media whenever possible. A bit-stream image preserves the entire storage space, including unallocated sectors, slack space, and corrupted blocks.
- **Hardware Protection:** Surveillance hard drives from active installations often experience heavy mechanical wear from 24/7 write operations. Repetitive sector-by-sector carving directly on degrading physical media risks drive head failure. Analysis on an image file eliminates physical drive stress.

---

## 3. Scope Decision: MVP Image Ingestion vs Future Live Acquisition

- **Locus MVP Scope Boundary:** The Locus MVP strictly ingests **pre-acquired forensic disk images** (`.dd`, `.raw`, `.img`).
- **Physical Drive Acquisition:** Upstream physical drive acquisition (connecting raw drives through write-blockers to create image files) is categorized as a **Phase 3 Roadmap Feature**.
- **Hardware Write-Blocking vs Software Read-Only Mode:**
  > [!IMPORTANT]
  > Opening a pre-acquired `.dd` file read-only in software prevents application-level write modifications to that image. It is **not** equivalent to a hardware write-blocker used during physical drive seizure and imaging. Physical write-blocking is an essential upstream hardware requirement.

---

## 4. Engineering Architecture: Open-Source Integrations

In digital forensics engineering, robust platforms coordinate proven components:
- **Image Ingestion & Hashing:** Python `hashlib` with 64 KB block streaming reader (dual SHA-256 and MD5 calculation).
- **Video Remuxing Engine:** `FFmpeg` / `PyAV` (packages raw H.264/H.265 elementary streams into standard `.mp4` containers in stream-copy mode without transcoding).
- **Secondary AI Analytics Engine:** Local `ONNX Runtime` executing `YOLOv8` models for candidate object and motion detection triage.
- **Transactional Database:** `SQLite` (high-speed transactional indexing of Master Sector Maps, camera channels, and provenance records).

### Core Custom Forensic Value in Locus:
1. **Multi-Vendor Proprietary Filesystem & Layout Adapters:** The modular parser framework detecting Dahua, Hikvision, CP Plus, and generic DVR storage structures.
2. **De-Interleaving & Metadata-Guided Carving:** Reassembling fragmented, interleaved multi-camera sector streams across circular recording buffers into continuous video streams.
3. **Forensic Integrity & UTC Normalization:** Managing raw timestamp preservation, non-destructive UTC calibration, and cryptographic provenance sidecars.
