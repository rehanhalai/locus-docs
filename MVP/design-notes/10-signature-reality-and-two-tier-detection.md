# Step 4.3 Specification: The Grounded Reality of DVR Signatures & Two-Tier Detection

**Back to [[MVP/MVP|MVP]]**

---

## 1. The Critical Reality: Do all vendors have unique codes on every sector?

**NO. That is a dangerous assumption.**

Surveillance manufacturers handle storage in fundamentally different ways:

### Category A: Frame-Wrapped Container Filesystems (Dahua / Validated DHAV Profiles)
- **Filesystem / Container:** **DHFS (Dahua File System, e.g., DHFS 4.1) / DHAV Containers**
- **Observed Structure:** For specific validated DHAV profiles (e.g., Dahua NVR4xxx series), observed evidence wraps video frames in proprietary binary headers (such as 32-byte structures starting with `DHAV` `0x44 0x48 0x41 0x56` and ending with `dhav` `0x64 0x68 0x61 0x76`). Header layout is model, firmware, and profile dependent and requires validation.
- **Detection:** Readily identified at the sector level on matching profiles because `DHAV` headers repeat across active channel streams.
- **CP Plus Handling:** CP Plus is evaluated independently via `CPPlusAdapter`; only when CP Plus evidence matches a validated DHAV profile is the DHAV parser activated.

### Category B: Index-Mapped Filesystems (Hikvision / EZVIZ)
- **Filesystem Name:** **HKFS (Hikvision File System)**
- **How it works:** Hikvision does **NOT** stamp an ASCII signature like "HKFS" on every single video sector. Instead, it maintains a master index structure called **`HIKBTREE`** (a B+ Tree index) at the start of the partition. 
- The video sectors themselves contain raw H.264/H.265 video NAL units without a "Hikvision" tag on every frame.
- **Detection:** Locus inspects partition metadata areas for the master index signatures `HIKBTREE`, `HKFS`, or `HIKB`.

### Category C: Circular Buffer & Generic Layouts (Swann / WFS / Generic OEMs)
- **Filesystem Name:** **WFS (WFS 0.4), ZHX, or Raw Streams**
- **How it works:** Uses a master superblock signature (e.g., `WFS 0.4`) at the partition header, while the remaining disk space is organized as a raw circular stream of multiplexed channels.

### Category D: Raw / Corrupted / Unindexed Video (Standard H.264/H.265 NAL Units)
- If the master index is deleted, overwritten, or uncharacterized, the data consists of raw **H.264 / H.265 Elementary Streams**.
- Standard candidate start codes: `0x00 0x00 0x00 0x01` followed by NAL type:
  - `0x67` / `0x27` (SPS - Sequence Parameter Set)
  - `0x68` / `0x28` (PPS - Picture Parameter Set)
  - `0x65` / `0x26` (IDR - Keyframe / I-Frame)

---

## 2. Two-Tier Detection Strategy in Locus

To maximize detection coverage without making unsupported assumptions, Locus utilizes a **Two-Tier Detection Strategy**:

```
                              [ Open Disk Partition ]
                                         │
                                         ▼
                 ┌───────────────────────────────────────────────┐
                 │  Tier 1: Master Index / Superblock Inspection │
                 │  (Reads partition start / metadata sectors)   │
                 └───────────────────────┬───────────────────────┘
                                         │
                 ┌───────────────────────┼───────────────────────┐
                 ▼                       ▼                       ▼
           [ "HIKBTREE" ]          [ "WFS 0.4" ]           [ "DHFS" ]
                 │                       │                       │
         Hikvision HKFS            Swann WFS               Dahua DHFS
         (B-Tree Traversal)      (WFS Parser)           (DHAV Parser)
                 │                       │                       │
                 └───────────────────────┼───────────────────────┘
                                         │ If Tier 1 Fails (No master index found)
                                         ▼
                 ┌───────────────────────────────────────────────┐
                 │  Tier 2: Frame-Level Sector Sampling Scan     │
                 │  (Samples sector boundaries across storage)   │
                 └───────────────────────┬───────────────────────┘
                                         │
                         ┌───────────────┴───────────────┐
                         ▼                               ▼
                 [ "DHAV" Bytes ]              [ "0x00000001" NAL ]
                         │                               │
                 Validated DHAV Stream            Candidate H.264 Stream
              (Parse validated headers)        (Carve SPS/PPS/IDR NALs)
```

---

## 3. Forensic Advantages & Safety Limits

1. **Catches Master Tables (Hikvision):** If a Hikvision disk index is intact, Locus parses the `HIKBTREE` index rapidly to obtain video boundaries and timestamps.
2. **Catches Validated Frame Containers (Dahua / CP Plus DHAV-compatible):** For validated profiles, Locus leverages repeating headers to map interleaved channels.
3. **Fallback for Damaged Disks:** If metadata indexes are overwritten or destroyed, Locus falls back to Tier 2 candidate NAL carving. Completeness and validity are determined by the presence of intact GOP structures; overwritten sectors remain unrecoverable.
