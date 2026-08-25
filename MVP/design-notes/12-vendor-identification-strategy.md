# Step 4.5 Specification: Unified Identification Strategy for Target Vendors

**Back to [[MVP/MVP|MVP]]**

---

## 1. Overview of the Strategy

To maximize detection coverage across known vendor formats without guessing, Locus uses a **Two-Phase Probing Engine**:

```
[ Forensic Image (.dd / .raw) ]
              │
              ▼
┌────────────────────────────────────────────────────────┐
│ Phase 1: Partition & Master Superblock Probe (Top-Down)│
│ Reads MBR/GPT and checks partition header signatures   │
└─────────────────────────────┬──────────────────────────┘
                              │
              ┌───────────────┴───────────────┐
              ▼                               ▼
       [ Signature Found ]            [ No Master Signature ]
              │                               │
       Candidate Profile                      ▼
 (Hikvision, Dahua, UNV, etc.)  ┌────────────────────────────────────────────────────────┐
                                │ Phase 2: Frame-Level Sector Sampling (Bottom-Up)       │
                                │ Samples 10,000 sector boundaries across the disk       │
                                └────────────────────────────┬───────────────────────────┘
                                                             │
                                              ┌──────────────┴───────────────┐
                                              ▼                              ▼
                                       [ DHAV Candidate ]             [ NAL Start Code ]
                                              │                              │
                                   DHAV-Compatible Profile         Candidate H.264/H.265
```

---

## 2. Phase 1: Master Superblock Probing (Top-Down)

When Locus opens the image, it parses Sector 0 (MBR) or Sector 1–33 (GPT) to find partition starting offsets. It then checks the first 1 MB of the video partition for specific Master Signatures:

| Target Vendor | Master Signature (Hex / ASCII) | Location Checked | Action on Match | Validation Status |
| :--- | :--- | :--- | :--- | :--- |
| **Hikvision** | `HIKBTREE` or `HKFS` or `HIKB` | Start of Video Partition (Sector 2048 / Offset 1MB) | Load `HikvisionHKFSAdapter` | Partially Validated |
| **Dahua Technology** | `DHFS` (e.g., `DHFS4.1` / `44 48 46 53`) | Superblock Offset 0x00 / Partition Start | Load `DahuaDHAVAdapter` | Partially Validated |
| **CP PLUS** | `DHFS` or `CP_PLUS` OEM Tag | Superblock Offset 0x00 | Load `CPPlusAdapter` | Researching *(Requires lab data)* |
| **Uniview (UNV)** | `UBIT` / `UVFS` / `UNV` | Partition Table Volume Descriptor | Load `UniviewAdapter` | Researching *(Requires lab data)* |
| **Honeywell** | `HNWL` or Hybrid GPT with `DHFS` | Partition Header & Config Table | Load `HoneywellAdapter` | Planned |
| **TP-Link (VIGI)** | `VIGI` / `TP-LINK` Volume Tag | GPT Partition Label / Superblock | Load `TPLinkAdapter` | Planned |

---

## 3. Phase 2: Frame-Level Sector Sampling (Bottom-Up Fallback)

If Phase 1 fails (because the DVR master index was zeroed, formatted, or uses raw unpartitioned storage), Locus falls back to **Sampling 10,000 Sector Boundaries**:

### Heuristic Scoring Matrix:

1. **Check for `DHAV` (`0x44 0x48 0x41 0x56`):**
   - If `DHAV` appears repeatedly at sector boundaries with matching `dhav` footers:
   - **Candidate: DHAV-compatible storage profile** (Score +100).
   - Validation hierarchy executed:
     $$\text{DHAV-like signature} \longrightarrow \text{Candidate Profile} \longrightarrow \text{Header Validation} \longrightarrow \text{Storage/Layout Validation} \longrightarrow \text{Metadata Validation} \longrightarrow \text{Device/Firmware Profile Validation}$$
   - *Note on Headers:* For a specific validated DHAV profile, observed video chunks may use a 32-byte header structure. Header layout is model, firmware, and profile dependent and requires validation.
   - *CP Plus Note:* A DHAV signature indicates candidate DHAV compatibility; it is **not sufficient alone to confirm CP Plus**. CP Plus profiles are routed through `CPPlusAdapter` for explicit profile validation. If validation passes, DHAV parsing routines are utilized; otherwise, status is set to `UNKNOWN`, `AMBIGUOUS`, or `UNSUPPORTED`.

2. **Check for Hikvision Frame Blocks (`HIKB` / Cluster Headers):**
   - If cluster headers match 2MB/16MB Hikvision block boundaries:
   - **Candidate: Hikvision HKFS cluster layout** (Score +100).

3. **Check for Standard H.264/H.265 NAL Units (`0x00 0x00 0x00 0x01`):**
   - Check if byte 4 is an SPS (`0x67` / `0x27`), PPS (`0x68` / `0x28`), or IDR Keyframe (`0x65` / `0x26`):
   - **Candidate H.264/H.265 stream evidence** (Uniview / TP-Link / Generic / Damaged Disk).
   - Additional validation required before recovery:
     - SPS/PPS parameter consistency (resolution, profile, framerate)
     - IDR and frame structure continuity
     - Codec parameter consistency across GOPs
     - Temporal ordering and timestamp monotonicity where available
     - Decoder test validation via reference decoder
   - *Important distinction:* **Candidate signature detection $\neq$ complete recording recovery.**

---

## 4. Strategy Resilience and Coverage

- **Fast Superblock Probing:** When partition tables and volume descriptors are intact, Phase 1 inspects headers rapidly without scanning the entire storage space.
- **Resilient Sector Sampling:** If the index was erased or damaged, Phase 2 examines raw sector header structures to detect candidate stream patterns or fall back to candidate H.264/H.265 stream carving.
