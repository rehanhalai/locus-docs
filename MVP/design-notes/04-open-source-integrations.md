# Open-Source Tools & Integration Architecture for Locus

**Back to [[MVP/MVP|MVP]]**

---

## 1. Disk Ingestion & Hashing (MVP vs Future Acquisition)

- **MVP Ingestion (Primary Scope):**
  - Uses Python `hashlib` with 64 KB block streaming readers (`open(filepath, 'rb')`).
  - Computes concurrent `SHA-256` and `MD5` cryptographic digests during ingestion without requiring secondary read passes.
- **Future Physical Imager Integration (Phase 3 Roadmap):**
  - **`dc3dd` / `dcfldd` (DoD Forensics):** Evaluated for future live physical drive acquisition modules when hardware write-blockers are connected directly to the workstation. Provides on-the-fly bad-sector zero-padding and live subprocess progress streaming.

---

## 2. Proprietary Filesystem & Header Parsing Research References

Surveillance vendors utilize proprietary storage layouts. Open-source forensic research projects provide foundational reference patterns:

- **`hikextractor` (Python):** Open-source reference parser for Hikvision DVR raw disk images, demonstrating partition boundary traversal and sector header unpacking.
- **`libHikvision` / `ezhikstract` (Python):** Reference implementations for parsing Hikvision/EZVIZ round-robin index files (`index00.bin`) and video cluster directories.
- **`dhav2mp4` / `dhav-extract`:** Open-source tools demonstrating how Dahua `DHAV` container headers wrap raw H.264 NAL units.

> [!NOTE]
> Open-source tools serve as reverse-engineering reference material. Locus implements a modular adapter architecture (`DahuaDHAVAdapter`, `HikvisionHKFSAdapter`, `CPPlusAdapter`) with explicit validation tracking and fallback hierarchies.

---

## 3. Video Remuxing & Frame Assembly (Feature 04)

- **`FFmpeg` / `PyAV` (Python bindings for FFmpeg C libraries):**
  - Wraps extracted raw H.264/H.265 Elementary Streams (`.h264`, `.h265`) into `.mp4` container wrappers.
  - Operates strictly in **stream copy mode (`-c:v copy`)** without transcoding (preserving original compressed bitstream payloads).
  - *Forensic Boundary:* PyAV/FFmpeg is **not** used to parse proprietary DVR storage structures or disk images; it is invoked solely on byte streams already extracted by Locus forensic parsers.

---

## 4. Video Analytics & Motion Detection (Feature 06)

- **`Ultralytics YOLOv8` via `ONNX Runtime`:** Local CPU/GPU inference engine for secondary object candidate tagging (`person`, `vehicle`).
- **OpenCV (`MOG2` / `HSV`):** Motion detection and dominant color estimation for analytical triage.
- *Forensic Boundary:* AI models perform secondary analytical triage and do not alter source bytes or establish legal identity.
