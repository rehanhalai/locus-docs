# Locus CCTV Forensics - Hardware & Vendor Compatibility Matrix

Locus provides specialized forensic acquisition, filesystem parsing, and zero-transcode video carving for CCTV DVRs, NVRs, and digital surveillance storage images.

---

## 📹 Supported DVR / NVR Ecosystems

| Vendor / Ecosystem | Common OEM Brands | File System / Signature | Container Format | Video Codecs | Status |
| :--- | :--- | :--- | :--- | :--- | :---: |
| **Xiongmai (XM) / HeimVision** | **HeimVision**, Xiongmai (XM), Jooan, BESDER, Hiseeu, Smonet, Anran, Jennov, Techage | FAT32 + `b"luo "` (0x6C 0x75 0x6F 0x20) | 4-Channel Multiplexed `FILE*.DAT` (8 MB blocks) | H.265 (HEVC), H.264 | ✅ **Fully Supported** |
| **Dahua Technology** | **Dahua**, CP PLUS (Orange Series), Lorex, Amcrest, Flir, Honeywell (some series) | `DHFS` (Dahua Harddisk FS) / `b"DHAV"` | 32-byte `DHAV` tagged sector frames | H.264, H.265 (HVC1) | ✅ **Fully Supported** |
| **Hikvision Digital Tech** | **Hikvision**, HiLook, EZVIZ, Annke, LTS, Swann (Pro Series), Trendnet | `HKFS` / `b"HIKB"` / `b"HKFS"` | 40-byte `HKFS` tagged sector frames | H.264, H.265, MPEG-4 | ✅ **Fully Supported** |
| **Swann / Asian Generic (WFS)** | **Swann**, KGuard, Zmodo, Night Owl, Q-See, Generic Asian DVRs | `WFS0.4`, `WFS1`, `WFS2`, `WFS3`, `WFS4` | Proprietary unallocated sector chunks | H.264, H.265 | ✅ **Fully Supported** |
| **Uniview (UNV)** | Uniview, ENS Security | UFS / Custom EXT4 Partition | MP4 / ES NAL Streams | H.265, H.264 | ✅ **Fully Supported** |
| **Standard SD & Dashcams** | Tapo (TP-Link), Wyze, VIOFO, Garmin, Nextbase, 70mai, Yi | FAT32 / exFAT / EXT4 | Standard `.mp4`, `.mov`, `.avi`, `.ts` | H.264, H.265, MJPEG | ✅ **Fully Supported** |

---

## 🛠️ Detailed Architecture Specifications

### 1. Xiongmai / HeimVision (`luo ` Format)
* **Storage Partitioning**: GPT or MBR containing a primary FAT32 volume.
* **Recording Layout**: Pre-allocated 8,388,608-byte (8 MB) `FILE0000.DAT` files located inside sequential directories (`DIR00001`, `DIR00002`...).
* **Header Structure**:
  * `0x00 - 0x03`: `b"luo "` (Magic identifier: `0x6C 0x75 0x6F 0x20`)
  * `0x04 - 0x07`: `uint32_le` Start Timestamp (UTC epoch seconds)
  * `0x08 - 0x0B`: `uint32_le` End Timestamp (UTC epoch seconds)
  * `0x0C - 0x0F`: `uint32_le` Camera 1 start byte offset
  * `0x10 - 0x13`: `uint32_le` Camera 2 start byte offset
  * `0x14 - 0x17`: `uint32_le` Camera 3 start byte offset
  * `0x18 - 0x1B`: `uint32_le` Camera 4 start byte offset
* **Extraction Strategy**: Fast FAT32 table indexing + zero-transcode HEVC/H.264 elementary stream demuxing into timestamp-synchronized MP4 containers.

---

### 2. Dahua / CP PLUS (`DHAV` Format)
* **Storage Partitioning**: Raw partition or custom MBR with `DHFS` filesystem.
* **Header Structure**: 32-byte frame headers starting with ASCII `b"DHAV"`.
* **Fields**: Channel ID (0-indexed), Timestamp (BCD encoded year/month/day/hour/min/sec), Frame Type (I-frame / P-frame / Audio), and Length.
* **Extraction Strategy**: Sector-aligned `DahuaHeaderUnpacker` parsing + zero-transcode packaging.

---

### 3. Hikvision (`HKFS` / `HIKB` Format)
* **Storage Partitioning**: `HKFS` master partition without standard OS boot records.
* **Header Structure**: 40-byte sector tags starting with `b"HKFS"` or `b"HIKB"`.
* **Fields**: Channel Number, Timestamp, Frame Type, Payload Sector Size.
* **Extraction Strategy**: Sector-aligned `HikvisionHeaderUnpacker` parsing.

---

### 4. WFS Generic (`WFS` Format)
* **Storage Partitioning**: Swann / Asian CCTV proprietary unpartitioned disks.
* **Header Structure**: Magic signatures `WFS4`, `WFS3`, `WFS2`, `WFS1`.
* **Extraction Strategy**: `WFSHeaderUnpacker` master directory index lookup.

---

## 💽 Supported Evidence Image Formats

| Forensic Image Format | Extension | Notes |
| :--- | :--- | :--- |
| **Raw Disk Image** | `.dd`, `.raw`, `.img`, `.bin` | Direct bitstream forensic clone |
| **Expert Witness Format** | `.E01`, `.E02`, `.E03`... | EnCase compressed forensic image (via `ewfmount` virtual raw mount) |
| **Virtual Disk Mounts** | `ewf1` | Zero-storage virtual raw mount |
| **Optical / Media** | `.iso`, `.001` | Optical media backup and split DD chunks |
| **Standalone CCTV Exports** | `.dav`, `.mp4`, `.h264`, `.h265`, `.264` | Native single-channel CCTV clips |
