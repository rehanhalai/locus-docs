# Feature 02: Multi-Tier Device & Storage Layout Identification

**Back to [[MVP/MVP|MVP]]**

---

## 1. Identification Pipeline Architecture

Device identification in Locus treats magic signatures as **initial candidate clues, not definitive forensic proof**. 

A signature scan alone is insufficient to identify drive structure. A multi-tier validation pipeline verifies signature context, header layout, partition structures, and metadata consistency before assigning vendor confidence:

```
Raw Evidence Image (.dd)
    │
    ▼
[Tier 1: Sector Signature Scanner]
- Sector-aligned read across initial 100 MB.
- Match candidate magic signatures (e.g., "DHAV", "HKFS").
    │
    ▼
[Tier 2: Candidate Identification]
- Register candidate vendor profiles (Dahua, Hikvision, etc.).
    │
    ▼
[Tier 3: Header Structure Validation]
- Unpack and validate sector block header fields (length, magic, checksum).
    │
    ▼
[Tier 4: Storage Layout & Partition Analysis]
- Verify MBR/GPT partition tables and raw sector ring-buffer bounds.
    │
    ▼
[Tier 5: Metadata & Channel Verification]
- Validate channel IDs, stream types (H.264/H.265), and timestamp monotonicity.
    │
    ▼
[Tier 6: Confidence Assignment & Adapter Selection]
- Assign confidence: KNOWN, LIKELY, UNKNOWN, UNSUPPORTED, or AMBIGUOUS.
- Select validated adapter (e.g., DahuaDHAVAdapter, HikvisionHKFSAdapter).
```

---

## 2. Distinction of Storage Attributes

Locus strictly separates distinct device and format parameters:

| Parameter | Example Value | Description |
| :--- | :--- | :--- |
| **Vendor** | Dahua Technology | Device manufacturer |
| **Model** | NVR4216-4KS2 | Hardware model series |
| **Firmware** | V4.000.0000001.0 | Device firmware version profile |
| **Storage Layout**| Raw Ring-Buffer | Physical disk sector allocation schema |
| **Filesystem** | DHAV Index Structure | File indexing mechanism |
| **Recording Format**| `.dav` / DHAV Wrapper | Sector wrapper container format |
| **Video Codec** | H.264 / High Profile | Elementary video bitstream encoding |

---

## 3. Explicit Confidence Classification Framework

If evidence parameters are contradictory or unverified, Locus assigns explicit confidence classifications without forcing a false positive match:

- **`KNOWN`:** Validated magic bytes + matching header checksum + valid partition map + verified channel metadata.
- **`LIKELY`:** Validated magic bytes + matching header structure, but partial index metadata.
- **`UNKNOWN`:** Unrecognized signatures or malformed sector headers. Vendor classification set to `UNKNOWN`.
- **`UNSUPPORTED`:** Recognized vendor signature, but firmware profile is explicitly flagged as unsupported.
- **`AMBIGUOUS`:** Sector signatures match multiple competing vendor definitions (e.g., Dahua OEM derivatives without clear firmware identifiers).

---

## 4. Scoped Binary Header Profile Schema

For every binary header format claim, Locus maintains a scoped profile block:

```yaml
vendor_profile:
  vendor: "Dahua Technology"
  model_series: "NVR4xxx / HCVR5xxx"
  firmware_profile: "V3.210+ / V4.0+"
  observed_format: "DHAV 32-byte sector header"
  evidence_source: "Laboratory test image dahua_nvr4216_500gb.dd"
  validation_status: "PARTIALLY_VALIDATED"
  known_limitations: "Requires 512-byte sector alignment; bad sector zero-padding required on damaged drives."
```

---

## 5. API Response Schema (`POST /api/device/identify`)

```json
{
  "status": "IDENTIFIED",
  "evidence_id": "EVD-2026-001",
  "candidate_vendor": "Dahua Technology",
  "model_series": "NVR4xxx Series",
  "confidence_level": "LIKELY",
  "detected_layout": "DHAV Sector Index",
  "partition_offset_sectors": 2048,
  "sector_size_bytes": 512,
  "detected_channels": [1, 2, 3, 4],
  "magic_signature_hex": "44484156",
  "selected_adapter": "DahuaDHAVAdapter",
  "adapter_status": "PARTIALLY_VALIDATED"
}
```
