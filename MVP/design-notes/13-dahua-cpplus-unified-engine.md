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

**Decision:** Locus will use a single **DHFS / DHAV Engine** to parse both Dahua and CP PLUS devices.

### Key Benefits:
1. **Zero Code Duplication:** We do not need to write separate parsers for Dahua and CP PLUS.
2. **Massive Market Coverage:** This single engine covers the vast majority of Indian retail and SMB installations (CP PLUS) as well as global Dahua systems.
3. **Automatic Rebranding Support:** Any other Dahua OEM brand (e.g., Amcrest, Lorex, Q-See) will also work out-of-the-box on this exact same engine.
