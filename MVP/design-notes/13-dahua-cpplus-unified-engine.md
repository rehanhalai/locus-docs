# Decision Note: Unified Dahua & CP PLUS Engine

*This document confirms the OEM relationship and unified engine architecture for Dahua Technology and CP PLUS.*

---

## 1. The Dahua ↔ CP PLUS OEM Relationship

- **Background:** CP PLUS (a leading surveillance brand in India) historically partners with and OEMs hardware and firmware from **Dahua Technology**.
- **Filesystem Identity:** CP PLUS DVRs and NVRs format storage media using Dahua's proprietary **DHFS (Dahua File System)** architecture.
- **Frame Packaging:** Video on CP PLUS drives is packaged in the **DHAV container format**:
  - Magic Header: `0x44 0x48 0x41 0x56` (`DHAV`)
  - Magic Footer: `0x64 0x68 0x61 0x76` (`dhav`)
  - Metadata layout: Identical channel ID byte offsets, frame type flags, payload length fields, and timestamp encoding.

---

## 2. Engineering Decision: The "DHFS Unified Engine"

**Decision:** Locus will use a primary **DHFS / DHAV Engine** for Dahua and DHAV-compatible CP PLUS devices, accessed through a `CPPlusAdapter` layer.

### Key Benefits:
1. **Zero Code Duplication:** We do not need to write separate parsers for Dahua and CP PLUS DHAV streams.
2. **Massive Market Coverage:** This single engine covers the vast majority of Indian retail and SMB installations (CP PLUS) as well as global Dahua systems.
3. **Graceful Fallback:** The `CPPlusAdapter` validates the DHAV header structure. If non-DHAV headers (e.g., Xiongmai / XM or generic ES) are detected on non-Dahua OEM CP Plus models, the pipeline safely falls back to the generic H.264 NAL carver or flags the stream as `UNKNOWN` rather than forcing a corrupted parse.
