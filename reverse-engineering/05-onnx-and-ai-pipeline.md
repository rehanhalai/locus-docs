# 5. ONNX Runtime & Secondary AI Triage Pipeline

**Back to [[00-index]]**

---

## 1. Local ONNX Runtime Architecture

Locus uses **ONNX Runtime** (Open Neural Network Exchange) to execute lightweight computer vision models locally on investigator workstations without external cloud dependencies or heavy PyTorch installations.

### Architectural Benefits
1. **Lightweight Deployment:** Models exported to ONNX format (`yolov8n.onnx`) occupy 12 MB – 45 MB of storage.
2. **Local CPU Optimization:** Uses C++ vector instructions (AVX2/AVX-512) for efficient CPU execution on standard laptops.
3. **Execution Provider Flexibility:** Supports CPU Execution Provider, CUDA (NVIDIA GPU), and DirectML (AMD GPU) transparently.

---

## 2. Resource Requirements & Execution Bounds

| Component | Minimum Requirements (Laptop) | Recommended Workstation | Technical Rationale |
| :--- | :--- | :--- | :--- |
| **CPU** | Quad-Core (2.5 GHz+) | 8+ Core Workstation CPU | Handles background sector carving and frame sampling pools. |
| **RAM** | 8 GB (4 GB free memory) | 16 GB – 32 GB RAM | Caps peak memory utilization under 2.5 GB. |
| **Storage** | SSD Storage | High-Speed NVMe SSD | Fast sequential read throughput during hashing and parsing. |
| **GPU** | Integrated Graphics (CPU Mode)| NVIDIA RTX 3060+ (CUDA) | Accelerates batch frame inference during secondary triage. |

---

## 3. Secondary AI Triage Execution Scenarios

### Scenario A: Automated Frame Sampling
- **Workflow:** Carved video clips are sampled at 1 frame per second.
- **Execution:** ONNX Runtime processes sampled frames and outputs candidate bounding boxes and confidence scores.
- **Storage:** Metadata indexed in SQLite (`ai_detections` table).

### Scenario B: Parameterized Timeline Search
- **Workflow:** Investigator queries filtered detection events (*"Show candidate person detections on Channel 2"*).
- **Execution:** Fast parameterized SQL query retrieves indexed event records and populates timeline overlays.

### Scenario C: Human-in-the-Loop Verification
- **Workflow:** Investigator inspects candidate detections in the React UI gallery.
- **Execution:** Detections are manually flagged as `VERIFIED` or `REJECTED` by the operator.
