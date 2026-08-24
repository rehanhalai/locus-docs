# MVP Scope & Image Ingestion Decisions

*These notes track technical constraints and scope reductions for the Minimum Viable Product (MVP).*

---

## 1. MVP Scope Reduction (Legal vs. Technical)
- **Decision:** For the MVP phase, all strict legal compliance features (e.g., BSA 2023 Section 63 certificate generation, chain of custody auditing, Tomaso Bruno gap presumption warnings) are **out of scope**.
- **Focus:** The MVP is strictly focused on creating a working technical prototype capable of parsing, carving, and playing back the video data.

## 2. Technical Constraints from Real-World Practices
While legal features are paused, the following *technical* realities must be supported by the engine:

- **Clock Drift Computation:** The engine must support applying a linear time offset to extracted timestamps. The investigator needs a way to say "Shift all times by +5 minutes" if the DVR clock was wrong.
- **Motion Gaps vs. Overwritten Data:** To accurately represent the timeline, the parser cannot just look at video headers. It must eventually parse the DVR's internal system logs to differentiate between a camera that recorded nothing (motion trigger) and a camera whose footage was overwritten (ring buffer).
- **No Re-encoding (Remuxing Only):** Locus must extract the raw H.264/H.265 byte streams and re-mux them into an `.mp4` container using a tool like FFmpeg or PyAV. Re-encoding the video is strictly forbidden because it destroys the original pixel data and drastically slows down the software.

## 3. Image Ingestion (Step 2)
- **Pending Decision:** Does the MVP require an initial SHA-256 hash generation of the `.dd` file upon ingestion, or is hashing deferred until a later version to speed up prototyping?
