# Step 4.3 Specification: The Grounded Reality of DVR Signatures & Two-Tier Detection

*This document answers whether all DVR vendors use unique ASCII magic bytes and details the actual two-tier detection architecture required.*

---

## 1. The Critical Reality: Do all vendors have unique codes on every sector?

**NO. That is a dangerous assumption.**

Surveillance manufacturers handle storage in two fundamentally different ways:

### Category A: Frame-Wrapped Container Filesystems (Dahua / CP PLUS)
- **Filesystem Name:** **DHFS (Dahua File System, e.g., DHFS 4.1)**
- **How it works:** Dahua wraps *every single video frame* in a proprietary 32-byte header starting with `DHAV` (`0x44 0x48 0x41 0x56`) and ending with a footer `dhav` (`0x64 0x68 0x61 0x76`).
- **Detection:** Extremely easy to spot at the sector level because `DHAV` repeats millions of times across the entire disk.

### Category B: Index-Mapped Filesystems (Hikvision / EZVIZ)
- **Filesystem Name:** **HKFS (Hikvision File System)**
- **How it works:** Hikvision does **NOT** stamp an ASCII signature like "HKFS" on every single video sector. Instead, it maintains a master index table called **`HIKBTREE`** (a B+ Tree structure) at the start of the partition. 
- The video sectors themselves contain raw H.264 video NAL units without a "Hikvision" tag on every frame.
- **Detection:** Locus must look for the master index signature `HIKBTREE` or `HKFS` in the partition's metadata area.

### Category C: Circular Buffer & Generic Layouts (Swann / WFS / Generic Asian OEMs)
- **Filesystem Name:** **WFS (WFS 0.4), ZHX, or Raw Streams**
- **How it works:** Uses a master superblock signature `WFS 0.4` at the partition header, but the rest of the disk is a raw circular stream of multiplexed channels.

### Category D: Raw / Corrupted / Unindexed Video (Standard H.264 NAL Units)
- If the master index is deleted, overwritten, or from an unbranded DVR, the data is raw **H.264 / H.265 Elementary Streams**.
- Standard start codes: `0x00 0x00 0x00 0x01` followed by NAL type:
  - `0x67` (SPS - Sequence Parameter Set)
  - `0x68` (PPS - Picture Parameter Set)
  - `0x65` (IDR - Keyframe / I-Frame)

---

## 2. The Finalized Two-Tier Detection Architecture in Locus

To ensure 100% reliability without making false assumptions, Locus uses a **Two-Tier Detection Strategy**:

```
                              [ Open Disk Partition ]
                                         │
                                         ▼
                 ┌───────────────────────────────────────────────┐
                 │  Tier 1: Master Index / Superblock Inspection │
                 │  (Reads first 1–10 MB of the partition)      │
                 └───────────────────────┬───────────────────────┘
                                         │
                 ┌───────────────────────┼───────────────────────┐
                 ▼                       ▼                       ▼
           [ "HIKBTREE" ]          [ "WFS 0.4" ]           [ "DHFS" ]
                 │                       │                       │
         Hikvision HKFS            Swann WFS               Dahua DHFS
         (Use B-Tree Parser)     (Use WFS Parser)       (Use DHAV Parser)
                 │                       │                       │
                 └───────────────────────┼───────────────────────┘
                                         │ If Tier 1 Fails (No master header found)
                                         ▼
                 ┌───────────────────────────────────────────────┐
                 │  Tier 2: Frame-Level Carving & Signature Scan │
                 │  (Samples sectors across the storage area)    │
                 └───────────────────────┬───────────────────────┘
                                         │
                         ┌───────────────┴───────────────┐
                         ▼                               ▼
                 [ "DHAV" Bytes ]              [ "0x00000001" NAL ]
                         │                               │
                Dahua/CP PLUS Stream              Raw Generic H.264
             (Parse 32-byte headers)           (Carve SPS/PPS/IDR NALs)
```

---

## 3. Summary of Benefits

1. **Catches Master Tables (Hikvision):** If a Hikvision disk is intact, Locus reads the `HIKBTREE` index in milliseconds to get a complete list of videos with exact timestamps.
2. **Catches Frame Containers (Dahua):** If it's Dahua/CP PLUS, Locus uses the repeating `DHAV` headers to map channels.
3. **Failsafe for Damaged Disks:** If the index was wiped, Locus falls back to Tier 2 (raw H.264 NAL carving) and still recovers the video.
