# Detailed Deliverables & Roadmap (Early Target: Sept 10–12 | Hard Cutoff: Sept 20)

**Back to [[locus]]**

---

## Strategic Goal: Early Portal Submission (Target: Sept 10–12)

- **Early Target Submission Window:** September 10 – 12, 2026 (To submit as soon as college SPOC credentials arrive & avoid last-minute portal crashes)
- **Hard Final SIH Cutoff:** September 20, 2026

---

## Accelerated Task Checklist

### Phase 1: Aug 23 – Aug 30 (Team Lock & Monorepo Skeleton)

- [ ] **Team & Organization**
  - [ ] Pitch and recruit Full-Stack Developer (React + FastAPI + SQLite).
  - [ ] Pitch and recruit Python Systems/Forensics Developer (Binary file carving + PyAV).
  - [ ] Verify SIH mandatory female representation requirement is satisfied.
  - [ ] Assign explicit team roles (Architect, Full-Stack, Python Systems, AI/CV, UI/UX, Docs & QA).
- [ ] **Repository & Monorepo Setup**
  - [ ] Initialize Git repository `locus` with monorepo structure (`/desktop`, `/frontend`, `/backend`, `/shared`).
  - [ ] Configure root `package.json` runner scripts (`npm run dev`) for concurrent launching.
  - [ ] Create Python virtual environment (`venv`) with `requirements.txt` (`fastapi`, `uvicorn`, `pyav`, `opencv-python`, `onnxruntime`).
  - [ ] Initialize React + TypeScript + Vite frontend with Tailwind CSS and ShadcnUI.
- [ ] **Data & Database Foundation**
  - [ ] Create synthetic 200 MB sample DVR raw disk dump (`sample_dahua.dd`) for fast everyday development testing.
  - [ ] Design SQLite database schema (`cases`, `evidence_files`, `carved_clips`, `timeline_events`, `hash_logs`, `audit_trail`).
  - [ ] Build baseline FastAPI server boilerplate with `/health`, `/api/cases`, `/api/ingest` endpoints.

---

### Phase 2: Aug 31 – Sept 6 (Core MVP: Carving Engine & UI Dashboard)

- [ ] **Python Acquisition, Carving & Remuxing**
  - [ ] Integrate `dc3dd` subprocess for live physical drive acquisition with bad-sector zero-padding.
  - [ ] Implement read-only file handler (`open(filepath, 'rb')`) with baseline SHA-256 & MD5 hash calculation on ingestion.
  - [ ] Build Dahua (`DHAV` magic header) sector scanner to identify frame boundaries and raw timestamps.
  - [ ] Build Hikvision sector scanner pattern matching logic for proprietary headers.
  - [ ] Integrate PyAV / FFmpeg remuxing function to convert carved H.264/H.265 streams into web-playable `.mp4` clips.
- [ ] **Electron Shell & React Dashboard UI**
  - [ ] Configure Electron `main.js` launcher to automatically spawn Python FastAPI backend process on app start.
  - [ ] Build Case Intake screen with native OS desktop file picker (`dialog.showOpenDialog`).
  - [ ] Build Evidence Registry table displaying raw file path, file size, baseline SHA-256 hash, and status badge.
  - [ ] Build Single-Camera HTML5 video player component with play/pause, speed controls, and frame-by-frame stepping.
- [ ] **API & WebSocket Integration**
  - [ ] Connect FastAPI REST endpoints to React frontend via Axios/Fetch.
  - [ ] Setup WebSocket endpoint (`/ws/progress`) to stream live carving progress percentages to the UI.

---

### Phase 3: Sept 7 – Sept 11 (AI Analytics, Timeline, PPT & Video - READY TO SUBMIT!)

- [ ] **Local AI Analytics (ONNX Runtime)**
  - [ ] Convert/download pre-trained YOLOv8 model in ONNX format (`yolov8n.onnx` ~12 MB).
  - [ ] Implement Python frame sampler (extracting 1 frame per second from carved `.mp4` clips).
  - [ ] Execute ONNX Runtime inference to detect `person`, `vehicle` (car/motorcycle/truck), and `object` targets.
  - [ ] Write AI detection labels, confidence scores, and frame timestamps into SQLite `timeline_events` table.
- [ ] **Multi-Camera Synchronized Timeline & Reports**
  - [ ] Build interactive multi-camera master timeline component & filter query bar.
  - [ ] Implement automated PDF report generator (`GET /api/cases/{id}/export-pdf`).
- [ ] **SIH Submission Package (EARLY TARGET REACHED!)**
  - [ ] **SIH Presentation PPT:** Design official presentation slides (Problem Statement, Solution Architecture, OEM Analysis, Tech Stack, Innovation).
  - [ ] **Demo Video:** Record a clean 2–3 minute prototype video walkthrough of Locus operating.
  - [ ] **EARLY PORTAL SUBMISSION:** Submit proposal as soon as college SPOC login credentials arrive!

---

### Phase 4: Sept 12 – Sept 20 (Buffer & Polish)

- [ ] Polish cross-platform build execution on Windows (`.exe`) and Linux (`.AppImage`).
- [ ] Conduct extended testing on larger DVR disk dump samples.
- [ ] Final fallback submission buffer window if college SPOC delays login access.
