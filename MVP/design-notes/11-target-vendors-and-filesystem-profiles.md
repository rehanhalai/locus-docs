# Step 4.4 Specification: Target Vendor Profiles (The Big 6)

*This document defines the 6 core surveillance vendors targeted by Locus MVP and maps their proprietary filesystem structures and signatures.*

---

## 1. The Big 6 Target Vendors

Targeting these 6 vendors covers **>85% of real-world surveillance infrastructure** (especially in the Indian market targeted by NTRO/SIH):

| Vendor | Primary Market Tier | Core Filesystem / Structure | Signature / Identifier |
| :--- | :--- | :--- | :--- |
| **1. Dahua Technology** | Global Commercial / Enterprise | **DHFS (e.g. DHFS 4.1)** | `DHAV` header (`0x44484156`), `dhav` footer (`0x64686176`) |
| **2. CP PLUS** | #1-2 Volume in India / SMB | **DHFS / Custom OEM** | Shared Dahua `DHAV` architecture or CP Plus raw stream headers |
| **3. Hikvision** | Global / Enterprise / Smart City | **HKFS (Hikvision FS)** | Master `HIKBTREE` index, cluster metadata blocks, raw NAL payloads |
| **4. Uniview (UNV)** | Enterprise / Campus / Govt | **UNV-FS / Custom Linux** | Master partition descriptors, raw TS/H.264 stream blocks |
| **5. Honeywell Security** | Banking / High-Security Industrial | **Honeywell / OEM Hybrid** | Encapsulated DAV/MP4 containers on custom Linux partitions |
| **6. TP-Link (VIGI / Tapo)** | Modern Cloud/NVR & SMB | **VIGI / FAT-Ext Hybrid** | Pre-allocated MP4/TS circular chunks with `index.dat` / `vigi` tags |

---

## 2. Technical Profile Mapping

### A. The Dahua & CP PLUS Family (DHFS Engine)
- **Relationship:** CP PLUS hardware extensively uses Dahua OEM architecture for its firmware and storage layouts.
- **Parsing Strategy:** Frame-level validation. Locus checks for the 32-byte `DHAV` header to extract the 1-byte channel number and 4-byte POSIX timestamp.

### B. Hikvision (HKFS Engine)
- **Parsing Strategy:** Master Index Traversal. Locus parses the `HIKBTREE` located in the metadata area to map logical camera files directly to physical disk clusters.

### C. Uniview & Honeywell (Enterprise Hybrid Engines)
- **Parsing Strategy:** Partition-level superblock detection with sector-aligned H.264 stream segmentation.

### D. TP-Link VIGI (Modern Stream Engine)
- **Parsing Strategy:** Structured circular chunk indexing, parsing embedded H.264/H.265 NAL units and camera metadata tracks.

---

## 3. Universal Fallback: The Generic H.264/H.265 NAL Carver
- If a drive from one of these vendors has a completely destroyed or encrypted index structure, Locus falls back to its **Universal NAL Carver** (`0x00000001` start codes), recovering raw I-Frames and P-Frames regardless of the brand.
