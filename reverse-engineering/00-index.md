# Reverse Engineering & Technical Foundations (Locus)

This section provides technical background notes on CCTV storage layouts, binary stream parsing, media remuxing boundaries, cryptographic integrity, and local AI triage execution.

---

### 📚 Table of Contents

1. [[01-cctv-and-dvr-basics]] — Overview of surveillance ring-buffer storage layouts vs standard operating system filesystems.
2. [[02-parsing-and-carving]] — Sector header decoding, Master Sector Map construction, and metadata-guided stream recovery.
3. [[03-video-processing-and-ai]] — PyAV/FFmpeg zero-transcoding remuxing boundaries and secondary AI triage search pipelines.
4. [[04-forensics-and-hashing]] — Cryptographic hashing (`SHA-256`, `MD5`), provenance tracking, and processing audit logs.
5. [[05-onnx-and-ai-pipeline]] — Local CPU/GPU ONNX Runtime execution model (YOLOv8) and resource boundaries.
