# Step 8 Specification: Secondary AI Analytics & Motion Triage

**Back to [[MVP/MVP|MVP]]**

---

## 1. Secondary AI Scoping & Purpose

In Locus, AI Analytics operates strictly as a **secondary analytical/triage tool** after read-only evidence ingestion, layout parsing, and stream validation are complete.

- **Primary Objective:** Accelerate investigator search by indexing candidate object (`person`, `vehicle`) and motion events.
- **Forensic Boundary:** AI detection outputs are candidate tags requiring human investigator review (`VERIFIED`, `REJECTED`, `UNREVIEWED`). AI outputs cannot establish legal authenticity or replace cryptographic hashing.

---

## 2. Local Air-Gapped Inference Pipeline

- **Local Execution:** 100% offline local inference using ONNX Runtime (`yolov8n.onnx`). No cloud API dependencies.
- **Motion Pre-Filtering:** OpenCV MOG2 background subtraction filters out motion voids (static scenes) to conserve CPU/GPU resources.
- **Metadata Indexing:** Bounding boxes, confidence scores, and frame timestamps are stored in SQLite (`ai_detections` table).

---

## 3. Human-in-the-Loop Review & Timeline Navigation

1. **Timeline Overlays:** Search results populate candidate tick marks on the interactive React timeline.
2. **Search Gallery:** Investigators filter candidate detections by class, confidence, and timestamp.
3. **Seek Navigation:** Selecting a candidate detection seeks the master video playhead to the target frame for human verification.
