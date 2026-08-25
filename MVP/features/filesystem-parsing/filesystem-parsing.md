# Feature 03: Storage Layout & Vendor Structure Parsing

**Back to [[MVP/MVP|MVP]]**

---

## 1. Storage Layout Parsing Principles

Standard operating systems expect partitions to conform to known filesystem standards (FAT32, NTFS, EXT4). Surveillance DVRs frequently bypass standard filesystems, storing raw video frames across custom sector layouts.

The Storage Layout & Structure Parsing module reads drive partitions and proprietary vendor indices to map sector locations before video carving begins.

---

## 2. Sector Header Variations Across Vendors

Header formats vary significantly across OEM manufacturers and firmware revisions. Locus strictly scopes parsing logic to laboratory-validated format profiles rather than assuming a universal structure:

| Vendor Profile | Container / Wrapper | Header Layout Description | Validation Status |
| :--- | :--- | :--- | :--- |
| **Dahua DHAV** | `.dav` / DHAV | For validated profiles (e.g., NVR4xxx series), observed 32-byte sector wrapper containing `DHAV` magic signature, channel ID, timestamp, payload length, and frame type. | **Partially Validated** |
| **Hikvision HKFS** | HKFS Block | HKFS partition table index mapping block clusters, channel numbers, and absolute sector offsets. | **Partially Validated** |
| **CP Plus** | DHAV derivative | Custom DHAV variant header; requires laboratory firmware validation. | **Researching (Phase 2 — not in MVP)** *(Requires lab data)* |
| **Generic Fallback**| Raw ES Stream | Sector-aligned H.264/H.265 NAL unit start-code scanner (`0x00000001`). | **Validated Fallback** |

> [!WARNING]
> Header parsing parameters (magic bytes, struct packing masks) are version-controlled per adapter. Unrecognized header variations are explicitly logged as `UNSUPPORTED_HEADER` rather than forcefully misparsed.

---

## 3. Master Sector Map Schema (`sector_map`)

Header parsing builds a Master Sector Map in SQLite (`sector_map`), recording frame locations across disk sectors:

| Column Name | Data Type | Sample Value | Description |
| :--- | :--- | :--- | :--- |
| `id` | `INTEGER` (PK) | `8201` | Record primary key |
| `evidence_id` | `TEXT` (FK) | `"EVD-2026-001"` | Source evidence image ID |
| `sector_offset` | `INTEGER` | `1048576` | Absolute byte offset in disk image |
| `sector_length` | `INTEGER` | `65536` | Sector block length in bytes |
| `channel_id` | `INTEGER` | `1` | Discovered camera channel ID |
| `raw_timestamp` | `TEXT` | `"2026-08-25 14:22:05"` | Unmodified raw timestamp string |
| `frame_type` | `TEXT` | `"I-FRAME"` | Keyframe vs Delta frame flag |
| `parser_status` | `TEXT` | `"VALIDATED"` | `VALIDATED`, `AMBIGUOUS`, or `CORRUPTED` |

---

## 4. API Specification (`POST /api/layout/parse`)

- **Request Payload:**
  ```json
  {
    "evidence_id": "EVD-2026-001",
    "adapter_name": "DahuaDHAVAdapter"
  }
  ```
- **Response Payload:**
  ```json
  {
    "status": "COMPLETED",
    "evidence_id": "EVD-2026-001",
    "sectors_mapped": 14200,
    "channels_discovered": [1, 2, 3, 4],
    "first_raw_timestamp": "2026-08-25T14:00:00Z",
    "last_raw_timestamp": "2026-08-25T18:30:00Z",
    "unparsed_sectors_count": 12
  }
  ```
