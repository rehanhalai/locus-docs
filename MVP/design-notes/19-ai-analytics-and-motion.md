# Step 8 Specification: AI-Assisted Analytics & Motion Detection

*This document outlines the architecture for applying offline, privacy-first computer vision to carved forensic video, transforming hours of raw footage into a highly searchable, indexed timeline.*

---

## 1. The "Needle in a Haystack" Problem

A standard 16-channel DVR recording over 30 days generates **480 days** of continuous video. If a suspect walked past a camera for exactly 4 seconds, finding that event manually requires an investigator to sit and fast-forward through hours of empty footage. 

**The Goal of Step 8:** To automatically scan the video in the background, detect relevant entities (Motion, Persons, Vehicles), and write their exact timestamps to an index. This allows the investigator to "search" the video like a text document.

---

## 2. Strict Offline AI Architecture Constraints

Due to the extreme sensitivity of criminal forensics and chain-of-custody rules, Locus adheres to a strict **Air-Gapped Privacy Model**:
- **No Cloud APIs:** We cannot send frames to OpenAI Vision, AWS Rekognition, or Google Cloud Vision.
- **Local Execution Only:** All AI models must run locally on the investigator's machine (using CPU, or GPU acceleration via CUDA/DirectML if available).
- **Engine Choice:** We use **ONNX Runtime** (Open Neural Network Exchange) to run highly optimized, quantized models (like YOLOv8-nano or YOLOv8-small) locally without requiring heavy PyTorch installations.

---

## 3. The Analytics Pipeline

The analytics engine runs in two complementary stages to maximize speed:

### Stage A: Motion Void Detection (DVR-Scan Logic)
Before running heavy object detection, we filter out dead time. 
- The system extracts frames at a low frame rate (e.g., 2 FPS).
- Applies background subtraction (e.g., MOG2) to detect structural pixel changes.
- If no motion is detected over a span of time, that segment is marked as a **Motion Void**.
- **Result:** We safely ignore hours of empty footage (like an empty parking lot at 3 AM), drastically reducing the workload for the AI model.

### Stage B: Object Detection (YOLOv8 / ONNX)
For segments where motion *is* detected, the system feeds frames into the YOLOv8-ONNX model.
- Scans for specific forensic classes: `Person`, `Car`, `Truck`, `Motorcycle`, `Backpack`.
- Generates bounding boxes and confidence scores (e.g., `Person: 88%`).
- These events are emitted to the SQLite database.

---

## 4. Database Schema (The Search Index)

We introduce a new table to store AI events. Crucially, we do *not* store duplicate video clips. We only store the *metadata* (the timestamps).

```sql
CREATE TABLE IF NOT EXISTS ai_analytics_events (
    event_id INTEGER PRIMARY KEY AUTOINCREMENT,
    channel_id INTEGER NOT NULL,
    start_timestamp INTEGER NOT NULL,
    end_timestamp INTEGER NOT NULL,
    event_type TEXT NOT NULL,         -- 'MOTION', 'PERSON', 'VEHICLE'
    confidence REAL,                  -- e.g., 0.85
    bounding_box TEXT,                -- JSON: [x, y, width, height]
    thumbnail_blob BLOB,              -- Optional: tiny base64 JPEG of the detection
    FOREIGN KEY(channel_id) REFERENCES camera_channels(channel_id)
);
```

---

## 5. The Investigator POV: Smart Timeline & Search

The data in the SQLite index powers a radically simplified user interface:
1. **The Heatmap Timeline:** The playback timeline displays color-coded tick marks (e.g., Red for Persons, Blue for Vehicles) indicating exactly where events occurred.
2. **"Skip Empty Time" Toggle:** The UI can automatically fast-forward through Motion Voids, playing only the segments with active events.
3. **The Search Bar:** An investigator can type "Person" and set a confidence threshold (e.g., >80%). A gallery of thumbnail detections appears. Clicking a thumbnail seeks the master video playhead to that exact timestamp.

---

## Appendix: Plain English Terminology

If the technical flow above sounds abstract, here is how you can explain the **Analytics Index** to a non-technical user or project manager:

### The 1,000-Page Book Analogy
Imagine you are given a 1,000-page book and told: *"Find every time the word 'Red Car' is mentioned."* If you do this manually, you have to read every single page. This is what investigators currently do with 30 days of CCTV footage—they sit there holding the fast-forward button.

**The Solution:** You need an **Index** at the back of the book. If you have an index, you just look up "Red Car," see it appears on pages 45, 102, and 890, and flip directly to those pages.

### How Locus Acts as the Indexer
1. **The Background Reader:** While the investigator is doing other paperwork, the Locus AI is "watching" the extracted video at super high speed in the background. 
2. **Writing the Index:** Every time the AI spots something interesting, it writes a tiny text entry into our database (our "Index at the back of the book"). It writes: *“At 04:15:30 PM on Camera 2, I saw a Person.”*
3. **The Search Bar:** When the investigator sits down, they don't have to watch the video. They just type "Person" into a search bar, see a list of timestamps, and click one. The video instantly jumps to the exact second the suspect walked on screen.
