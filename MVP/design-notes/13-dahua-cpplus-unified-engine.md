# Decision Note: Dahua and CP Plus Forensic Adapter Architecture

**Back to [[MVP/MVP|MVP]]**

---

## 1. Dahua & CP Plus OEM Format Relationship

CP Plus surveillance recorders in South Asia frequently utilize hardware and firmware licensed from Dahua Technology. Consequently, many CP Plus models format storage media using variants of Dahua's `DHAV` container structure (`0x44 0x48 0x41 0x56`).

However, from a digital forensics perspective:

> [!WARNING]
> **Forensic Rule on OEM Assumptions:**  
> **Locus does not blindly assume CP Plus = Dahua for all models and firmware versions.**  
> While CP Plus Cosmic/Orange series devices often match Dahua DHAV sector structures, specific CP Plus firmware profiles may introduce modified index tables or non-standard frame headers.

---

## 2. Adapter Architecture & Validation Status

Locus maintains separate modular adapters for Dahua and CP Plus to allow independent validation tracking:

- **`DahuaDHAVAdapter`:** Dedicated adapter for Dahua NVR4xxx / HCVR5xxx profiles.  
  - *Validation Status:* **Partially Validated** (Tested on laboratory `.dd` images).
- **`CPPlusAdapter`:** Dedicated adapter for CP Plus Cosmic series profiles (Phase 2 — not in MVP).  
  - *Validation Status:* **Researching** *(Requires additional laboratory model/firmware validation data)*.

---

## 3. Fallback Mechanism

If a CP Plus image exhibits standard `DHAV` magic signatures and matching header checksums, `CPPlusAdapter` delegates sector decoding to the validated DHAV parsing pipeline while recording `adapter_used = "CPPlusAdapter (DHAV Fallback)"` in artifact provenance records. If header parameters deviate, the adapter flags the layout as `AMBIGUOUS` or `UNSUPPORTED`.
