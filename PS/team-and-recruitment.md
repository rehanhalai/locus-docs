# Team Structure & Engineering Roles

**Back to [[locus]]**

---

## 1. Hackathon Team Guidelines

- **Team Size:** 6 student members
- **Diversity Requirement:** Minimum 1 female team member (SIH mandatory compliance)
- **Technical Competency:** Forensic storage research, systems programming, local AI execution, type-safe UI, technical documentation & validation QA.

---

## 2. Engineering Roles & Responsibilities

| Role # | Role Title | Core Focus Area | Primary Technical Stack |
| :---: | :--- | :--- | :--- |
| **1** | **Systems Architect & Team Lead** | Overall system architecture, security model, API contracts, database schema, SIH presentation strategy. | FastAPI, Python 3.11, SQLite 3, Monorepo Architecture |
| **2** | **Forensic Adapter & Systems Dev** | Binary disk image parsing, OEM magic header scanning, Dahua/Hikvision parser logic, PyAV remuxing. | Python, Binary I/O, Struct, PyAV, FFmpeg bindings |
| **3** | **Full-Stack Desktop Developer** | Case intake UI, native desktop file dialogs, REST/WebSocket API integration, SQLite transactional queries. | Electron, React 18, TypeScript, FastAPI, WebSockets |
| **4** | **Secondary AI & Computer Vision Dev**| ONNX Runtime pipeline integration, YOLOv8 object/motion candidate triage, frame sampling, confidence indexing. | Python, ONNX Runtime, YOLOv8, OpenCV, NumPy |
| **5** | **Frontend & UI/UX Specialist** | Multi-camera synchronized timeline matrix, hex sector viewer component, interactive video grid UI. | React, TypeScript, Vite, Tailwind CSS, HTML5 Canvas |
| **6** | **Forensic QA, Docs & Validation Lead** | Ground-truth dataset testing, hash parity validation, SOP documentation, Forensic Evidence Report templates. | Technical Writing, QA Benchmarking, SOPs, PDF Templates |

---

## 3. Recruitment & Project Alignment Pitch

> **Project Alignment Summary:**  
> **Locus** is a specialized desktop forensic platform for multi-vendor DVR/NVR surveillance disk image analysis, layout reconstruction, stream validation, timeline normalization, and artifact provenance.  
>  
> The project combines low-level binary disk parsing (Python / C-bindings), multi-camera timeline UI (React / TypeScript), and offline computer vision triage (ONNX Runtime / YOLOv8) within a local Electron desktop container.
