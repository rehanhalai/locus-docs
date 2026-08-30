# 🧠 Flow 06 & 07: Local AI Video Analytics (ONNX Runtime + YOLOv8 + Motion Gating) & Evidence Search

> **Module:** `backend/app/modules/analytics/`  
> **Status:** `✅ Completed (14/14 Flow 06 Tests Passing, 104/104 Overall)`  
> **Purpose:** Perform 100% offline, local AI video analytics over carved DVR video clips. Combines fast OpenCV MOG2 background-subtraction motion gating with YOLOv8 Nano ONNX Runtime inference to index forensic events (`person`, `car`, `truck`, `motorcycle`, `backpack`, `knife`, `motion`) into SQLite with calibrated timestamps and normalized bounding boxes.

---

## ⚡ The Two-Stage "Funnel" Architecture

```text
Carved .mp4 Video Clip
         │
         ▼
┌──────────────────────────────────────┐
│  Stage 1: Fast Motion Gating (MOG2)  │  --> Skips static empty frames (< 0.2ms / frame)
└──────────────────┬───────────────────┘
                   │ (Frames with motion)
                   ▼
┌──────────────────────────────────────┐
│  Stage 2: YOLOv8 ONNX Engine (CPU)   │  --> Detects Person, Vehicles, Weapons, Bags...
└──────────────────┬───────────────────┘
                   │
                   ▼
┌──────────────────────────────────────┐
│  Stage 3: SQLite TimelineEvent Table │  --> [10:04:12 AM] Person (94%) at [x,y,w,h]
└──────────────────┬───────────────────┘
                   │
                   ▼
┌──────────────────────────────────────┐
│  Stage 4: Sub-Second REST Search API │  --> Filter by Camera, Class, Confidence, Time
└──────────────────────────────────────┘
```

---

## 🗄️ Database Schema: `timeline_events` Table

```sql
CREATE TABLE timeline_events (
    id VARCHAR(64) PRIMARY KEY,                  -- e.g. "evt_a1b2c3d4e5f6"
    evidence_id VARCHAR(64) NOT NULL REFERENCES evidence_files(id),
    clip_id VARCHAR(64) REFERENCES carved_clips(id),
    camera_id INTEGER NOT NULL,
    timestamp DATETIME NOT NULL,                 -- Calibrated master forensic timestamp
    frame_number INTEGER NOT NULL DEFAULT 0,
    label VARCHAR(64) NOT NULL,                  -- "person", "car", "truck", "backpack", "knife", "motion"
    confidence FLOAT NOT NULL,                   -- e.g. 0.942
    bbox_x FLOAT NOT NULL DEFAULT 0.0,           -- Top-left X normalized (0.0 to 1.0)
    bbox_y FLOAT NOT NULL DEFAULT 0.0,           -- Top-left Y normalized
    bbox_w FLOAT NOT NULL DEFAULT 0.0,           -- Box Width normalized
    bbox_h FLOAT NOT NULL DEFAULT 0.0,           -- Box Height normalized
    is_motion BOOLEAN NOT NULL DEFAULT 1,
    created_at DATETIME NOT NULL
);
```

---

## 📡 REST Endpoints

| Endpoint | Method | Status Code | Description |
| :--- | :--- | :--- | :--- |
| `/api/v1/analytics/process` | `POST` | `202 Accepted` | Enqueues background motion gating and YOLOv8 inference over carved video clips. |
| `/api/v1/analytics/progress/{task_id}` | `GET` | `200 OK` (SSE) | Real-time Server-Sent Events stream broadcasting frame progress, event count, and percentage. |
| `/api/v1/analytics/events/{evidence_id}` | `GET` | `200 OK` | Fast parameter search & filtering by camera ID, object labels, minimum confidence, and time range. |

---

## 🧪 Test Verification (14 Flow 06/07 Tests / 104 Overall)

Verified across:
* `test_analytics_model.py`: Database table creation, foreign keys, cascade deletion, and `EventLabel` enum validation.
* `test_analytics_motion.py`: Background modeling, empty frame rejection, moving box detection, and model reset.
* `test_analytics_detector.py`: ONNX Runtime session loading, synthetic image inference, NMS suppression, and class filtering.
* `test_analytics_api.py`: Full end-to-end API processing lifecycle, SSE progress subscription, filtered search queries, and 404 security guards.
