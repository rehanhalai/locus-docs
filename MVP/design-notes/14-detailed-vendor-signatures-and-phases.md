# Detailed Guide: Vendor Signature Patterns, Detection Pipeline & Vendor Matrix

**Back to [[MVP/MVP|MVP]]**

---

## 1. Multi-Tier Vendor Signature Pipeline

In Locus, signature detection is a multi-step verification pipeline rather than a single byte check:

```
Raw Evidence Image (.dd)
    │
    ▼
[Phase 1: Sector Signature Scan]
- Fast sector-aligned read looking for candidate signatures ("DHAV", "HKFS", "UVFS").
    │
    ▼
[Phase 2: Header Structure Validation]
- Unpack sector block header fields (length, magic byte, checksum).
    │
    ▼
[Phase 3: Storage Layout & Partition Analysis]
- Verify partition tables (MBR/GPT) and raw ring-buffer allocation.
    │
    ▼
[Phase 4: Metadata Verification]
- Verify channel count, stream type (H.264/H.265), and timestamp monotonicity.
    │
    ▼
[Phase 5: Vendor Confidence Assignment]
- Assign status: KNOWN, LIKELY, UNKNOWN, UNSUPPORTED, or AMBIGUOUS.
```

---

## 2. Vendor Support & Adapter Matrix

| Vendor Name | Candidate Signatures | Format Profile | Adapter Name | Carving Status | Validation Status |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Dahua Technology** | `DHAV`, `DHFS` | DHAV sector wrapper / DHFS index | `DahuaDHAVAdapter` | Intact + Sector Carve | **Partially Validated** |
| **Hikvision** | `HKFS`, `HIKBTREE`, `HIKB` | HKFS B+ tree partition index | `HikvisionHKFSAdapter`| HKFS Index + Carve | **Partially Validated** |
| **CP Plus** | `DHAV` variant | CP Plus Cosmic series profile | `CPPlusAdapter` | DHAV Fallback | **Researching (Phase 2 — not in MVP)** *(Requires lab data)* |
| **Honeywell** | `HNWL`, `DHAV` | Hybrid EXT4 / Proprietary stream | `HoneywellAdapter` | Container Demux | **Planned** |
| **TP-Link** | `VIGI`, `TP-LINK` | Segmented circular recording buffer | `TPLinkAdapter` | Filesystem Parse | **Planned** |
| **Godrej** | Unknown | Proprietary OEM layout | `GodrejAdapter` | Generic Fallback | **Researching** *(Requires lab data)* |
| **Uniview** | `UBIT`, `UVFS`, `UNV` | UVFS volume descriptors | `UniviewAdapter` | Generic Fallback | **Researching** *(Requires lab data)* |
| **Matrix** | Proprietary | Custom partition schema | `MatrixAdapter` | Generic Fallback | **Researching** *(Requires lab data)* |

---

## 3. Generic Carving Fallback

If vendor signatures are unrecognizable or headers are destroyed, Locus falls back to generic H.264/H.265 NAL unit start code scanning (`0x00000001` + SPS `0x67` / IDR `0x65`).

Outputs from generic carving are explicitly flagged as `UNVALIDATED_CARVE` in metadata and reports to alert examiners to potential fragment misordering or missing timestamps.
