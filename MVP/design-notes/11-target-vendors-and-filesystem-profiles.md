# Step 4.4 Specification: Target Vendor Profiles (The Big 6)

**Back to [[MVP/MVP|MVP]]**

---

## 1. Target Vendor Profiles

Targeting these 6 vendor families covers an estimated majority of real-world surveillance deployments in India:

| Vendor | Primary Market Tier | Core Filesystem / Structure | Signature / Identifier | Validation Status |
| :--- | :--- | :--- | :--- | :--- |
| **1. Dahua Technology** | Global Commercial / Enterprise | **DHFS (e.g., DHFS 4.1)** | `DHAV` header (`0x44484156`), `dhav` footer | Partially Validated |
| **2. CP PLUS** | Volume Leader (South Asia / SMB) | **Custom OEM / DHAV Profile** | CP Plus OEM tags / DHAV variant | Researching (Phase 2 — not in MVP) *(Requires lab data)* |
| **3. Hikvision** | Global / Enterprise / Smart City | **HKFS (Hikvision FS)** | Master `HIKBTREE` index, cluster headers | Partially Validated |
| **4. Uniview (UNV)** | Enterprise / Campus / Govt | **UNV-FS / Custom Linux** | Master partition descriptors, UVFS tags | Researching *(Requires lab data)* |
| **5. Honeywell Security** | Banking / High-Security Industrial | **Honeywell / OEM Hybrid** | Encapsulated DAV/MP4 containers | Planned |
| **6. TP-Link (VIGI)** | Modern Cloud/NVR & SMB | **VIGI / FAT-Ext Hybrid** | Pre-allocated circular chunks with `vigi` tags | Planned |

---

## 2. Technical Profile Mapping & Parsing Strategies

### A. Dahua Technology (`DahuaDHAVAdapter`)
- **Observed Structure:** For validated Dahua NVR4xxx series images, frames are wrapped in proprietary binary headers (such as 32-byte headers starting with `DHAV`).
- **Parsing Strategy:** Frame header validation extracting channel ID, raw timestamp, and payload length to populate the Master Sector Map.

### B. CP PLUS (`CPPlusAdapter` — Phase 2, not in MVP)
- **Architecture:** Treated as an independent vendor family. When evidence matches a validated DHAV profile, `CPPlusAdapter` delegates decoding to the DHAV pipeline while documenting provenance lineage. If headers deviate, the adapter flags the layout as `AMBIGUOUS` or `UNSUPPORTED`.

### C. Hikvision (`HikvisionHKFSAdapter`)
- **Parsing Strategy:** Master Index Traversal. Locus parses the `HIKBTREE` located in partition metadata to map logical camera files directly to physical disk clusters.

### D. Uniview & Honeywell (`UniviewAdapter` / `HoneywellAdapter`)
- **Parsing Strategy:** Partition-level superblock detection with sector-aligned H.264/H.265 stream segmentation.

### E. TP-Link VIGI (`TPLinkAdapter`)
- **Parsing Strategy:** Structured circular chunk indexing, parsing embedded H.264/H.265 NAL units and camera metadata tracks.

---

## 3. Universal Fallback: Candidate H.264/H.265 NAL Carving
- If a disk image from one of these vendors has a destroyed or unindexed structure, Locus falls back to its **Candidate NAL Carver** (`0x00000001` start codes + SPS/PPS/IDR validation), extracting surviving video fragments. Extracted fragments are explicitly flagged as `UNVALIDATED_CARVE`.
