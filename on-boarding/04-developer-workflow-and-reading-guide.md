# 04 — Developer Workflow & Reading Roadmap

This guide covers the monorepo structure, local development workflow, safe testing techniques, and the recommended reading sequence for understanding the deep technical specs.

---

## 1. Monorepo Structure

```text
locus/
├── backend/                  # FastAPI Python Backend
│   ├── app/
│   │   ├── core/             # Configuration, DB session, binary locators
│   │   ├── db/               # SQLAlchemy models (Case, EvidenceFile, AuditLog)
│   │   └── modules/          # Core forensic engines
│   │       ├── acquisition/  # dc3dd subprocess execution & hashing
│   │       ├── identification/# Partition & OEM magic byte scanners
│   │       ├── carving/      # Sector parsing & PyAV remuxing
│   │       ├── timeline/     # Multi-camera sync & master clock
│   │       ├── analytics/    # OpenCV motion filter & ONNX YOLOv8
│   │       ├── search/       # Parameterized SQLite query engine
│   │       └── export/       # Zero-transcoding export & .sync.json sidecars
│   ├── bin/                  # Bundled standalone binaries (dc3dd for Linux/Windows)
│   └── tests/                # Automated pytest suite
├── frontend/                 # React UI (TypeScript + Vite + Tailwind CSS)
│   ├── src/
│   │   ├── components/       # UI components (multi-cam grid, timeline bar)
│   │   ├── pages/            # Case intake, analysis dashboard, export view
│   │   └── services/         # API & SSE client connections
└── desktop/                  # Electron Shell (main.js, preload.js)
```

---

## 2. Local Development Workflow

During active development, we run the frontend and backend in separate hot-reloading terminals:

### Running the Python Backend:
```bash
cd backend
# Activate virtual environment
source .venv/bin/activate  # (or .venv\Scripts\activate on Windows)

# Start FastAPI server with live reload on http://localhost:8000
uvicorn app.main:app --reload --port 8000
```
*Interactive API docs are available at `http://localhost:8000/docs`.*

### Running the React Frontend:
```bash
cd frontend
# Start Vite development server on http://localhost:5173
pnpm dev
```

---

## 3. How to Test Without Real CCTV Hardware

You do **not** need a real CCTV recorder or physical drive to test your code!

1. **Synthetic Disk Images:** 
   We generate small synthetic `.dd` / `.raw` files (10 MB to 50 MB) using Python or `dd` commands:
   ```bash
   dd if=/dev/urandom of=/tmp/mock_disk.raw bs=1M count=10
   ```
2. **`dc3dd` Flexibility:** 
   `dc3dd` treats files and block devices identically (`if=/tmp/mock_disk.raw` works the exact same way as `if=/dev/sdb`).
3. **Automated Pytest Suite:** 
   Run backend tests in seconds:
   ```bash
   cd backend
   pytest tests/
   ```

---

## 4. Recommended Documentation Reading Roadmap

To build deep mastery of the low-level binary specifications, read the documentation files in this order:

| Step | Document | What You Will Learn |
| :---: | :--- | :--- |
| **1** | [01-welcome-and-problem-statement.md](./01-welcome-and-problem-statement.md) | Problem statement & the 3 Golden Rules of Forensics |
| **2** | [02-core-features-and-analogies.md](./02-core-features-and-analogies.md) | The 8 core features explained via simple real-world analogies |
| **3** | [03-architecture-and-tech-stack.md](./03-architecture-and-tech-stack.md) | System architecture & technology selection rationale |
| **4** | [MVP/MVP.md](../MVP/MVP.md) | The complete MVP functional specification index |
| **5** | [MVP/development/01-acquisition-and-dc3dd-setup.md](01-acquisition-and-dc3dd-setup.md) | Why we bundle static binaries for disk imaging |
| **6** | [MVP/development/flow-02-device-identification.md](flow-02-device-identification.md) | Step-by-step device and filesystem discovery |
| **7** | [MVP/design-notes/14-detailed-vendor-signatures-and-phases.md](../MVP/design-notes/14-detailed-vendor-signatures-and-phases.md) | Deep breakdown of Dahua (`DHAV`), Hikvision (`HKFS`), and H.264 NAL units |
| **8** | [MVP/design-notes/18-timeline-synchronization-and-alignment.md](../MVP/design-notes/18-timeline-synchronization-and-alignment.md) | Math behind time offsets ($\Delta t$) and the 60 Hz master clock bus |

---

### Questions or Feedback?
If you spot an edge case or have ideas for optimization, discuss it with the team and add a note to the `MVP/development/` directory!
