# 5. ONNX Runtime & AI Execution Pipeline

**Back to [[00-index]]**

---

## 🤖 What is ONNX and Why Do We Need It?

**ONNX** stands for **Open Neural Network Exchange** (developed by Microsoft, Meta, and Amazon).

### The Problem without ONNX (The PyTorch Trap)
If we use standard **PyTorch** to run our YOLOv8 object detection AI model:
1. PyTorch requires installing heavy C++ CUDA libraries (`torch` & `torchvision`), which take up **5.5 GB to 8 GB of disk space**.
2. Packaging PyTorch into our `Locus.exe` desktop installer turns it into a **massive 8 GB download**.
3. PyTorch without a dedicated $2,000 NVIDIA GPU runs **very slowly on standard laptop CPUs**.

### The Solution with ONNX
1. **Model Export:** We convert our PyTorch YOLOv8 model (`.pt`) **once** into an optimized ONNX format (`.onnx`). The model size drops from 500 MB to just **12 MB – 45 MB**.
2. **Tiny Runtime:** Instead of 5 GB PyTorch, we install `onnxruntime` (a lightweight 15 MB C++/Python library).
3. **CPU Optimization:** ONNX Runtime uses hardware-level vector instructions (AVX2/AVX512 on Intel CPUs) to run inference **3x to 5x faster than PyTorch on standard laptop CPUs**.

---

## 💻 System Requirements Breakdown (Why these numbers?)

| Component | Minimum (8GB RAM / Quad-Core) | Recommended (16GB+ RAM / 8-Core) | Why it matters |
| :--- | :--- | :--- | :--- |
| **CPU** | Quad-Core i5 / Ryzen 5 | 8-Core i7/i9 or Ryzen 7/9 | Python carving scans raw disk sectors in parallel threads. More cores = faster carving. |
| **RAM** | 8 GB RAM | 16 GB – 32 GB RAM | Idle app uses ~600MB. Remaining RAM buffers raw disk dumps and video streams during extraction. |
| **Storage** | SSD (Solid State Drive) | High-speed NVMe SSD | Spinning HDDs read at 80 MB/s. NVMe SSDs read at 3,500 MB/s (carving finishes in seconds). |
| **GPU** | Integrated Graphics (CPU mode) | NVIDIA GPU (CUDA mode) | ONNX runs fast on CPU for small clips, but an NVIDIA GPU allows processing 200+ fps. |

---

## 🎬 AI Model Running Scenarios in Locus

### Scenario A: Automated Frame Sampling (On Carving)
- **Workflow:** When 10 minutes of video is carved, Locus extracts 1 frame per second (600 frames total).
- **Execution:** ONNX Runtime processes the 600 frames in a fast batch (~10–15 seconds on CPU).
- **Storage:** Detected objects (Person, Vehicle, Motion) are saved as JSON tags in the SQLite database.

### Scenario B: Instant Timeline Search (User Query)
- **Workflow:** Investigator searches: *"Show all vehicles between 2:00 PM and 3:00 PM"*.
- **Execution:** Locus does **not** rerun the AI! It executes a sub-second SQL query: `SELECT * FROM timeline_events WHERE object='vehicle' AND confidence > 0.8`.
- **Result:** Timeline scrub bar instantly places highlight markers at the exact timestamps.

### Scenario C: Facial Recognition Matching
- **Workflow:** Investigator uploads a suspect photo to match across all camera channels.
- **Execution:** Lightweight FaceNet ONNX model compares feature embeddings of the suspect photo against saved face thumbnails in SQLite.
- **Result:** Displays camera timeline matches ordered by similarity percentage (e.g., Camera 2 at 14:05:22 - 92% Match).
