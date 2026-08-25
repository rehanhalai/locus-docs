# Feature 06: Secondary AI Video Analytics & Human Review

**Back to [[MVP/MVP|MVP]]**

---

## 1. Primary AI Scoping & Forensic Boundaries

In Locus, Computer Vision AI (YOLOv8 via ONNX Runtime) is strictly scoped as a **secondary analytical/triage layer**. 

AI processing runs **only after** pre-acquired evidence disk images have been registered, cryptographically hashed, parsed, and validated.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       PRIMARY FORENSIC CORE LAYER                           │
│  Read-Only Image Ingestion ──► Dual SHA-256/MD5 ──► Layout Parse & Validation │
└──────────────────────────────────────┬──────────────────────────────────────┘
                                       │ Validated Video Clips (.mp4)
                                       ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                      SECONDARY AI ANALYTICAL TRIAGE LAYER                   │
│  1 FPS Frame Sampling ──► Local ONNX Runtime (YOLOv8) ──► Candidate Tags   │
└──────────────────────────────────────┬──────────────────────────────────────┘
                                       │ Candidate Bounding Boxes & Scores
                                       ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                     HUMAN-IN-THE-LOOP INVESTIGATOR REVIEW                   │
│  Investigator Verification ──► [VERIFIED] / [REJECTED] / [UNREVIEWED]       │
└─────────────────────────────────────────────────────────────────────────────┘
```

> [!CAUTION]
> **Forensic Boundaries of AI:**
> - AI **cannot** recover missing source bytes or reconstruct overwritten sectors.
> - AI **cannot** establish legal authenticity or replace cryptographic hash parity.
> - AI **cannot** repair incorrect evidence acquisition or corrupt sector headers.
> - AI **cannot** prove personal identity based on object detection alone.

---

## 2. AI Analytical Capabilities & Metadata Attributes

Locus separates distinct AI computer vision tasks into discrete analytical categories:

| AI Analytical Category | Engine / Model | Output Description | Forensic Label Language |
| :--- | :--- | :--- | :--- |
| **Object Detection** | YOLOv8n ONNX | Bounding boxes for candidate classes (`person`, `vehicle`). | `"AI-Generated Candidate Detection"` |
| **Face Detection** | UltraFace ONNX | Candidate facial bounding boxes in video frames. | `"AI-Generated Facial Candidate Bounding Box"` |
| **Motion Detection** | OpenCV MOG2 | Pixel-level temporal change detection. | `"Analytical Motion Region"` |
| **Color Classification** | OpenCV HSV Filter | Dominant hue range estimation for bounding boxes. | `"Estimated Target Color"` |

### Required Inference Metadata Attributes
Every AI detection record saved in SQLite (`ai_detections`) MUST include complete inference parameters:
- `model_name` (e.g., `"yolov8n.onnx"`)
- `model_version` (e.g., `"v8.0.200"`)
- `inference_engine` (e.g., `"ONNX Runtime CPU Provider"`)
- `target_class` (e.g., `"person"`)
- `confidence_score` (e.g., `0.875`)
- `frame_timestamp` (Raw and UTC Normalized)
- `bounding_box` (`[x, y, width, height]`)
- `source_artifact_id` (Parent clip record ID)
- `processing_version` (Locus AI pipeline version)

---

## 3. Human-in-the-Loop Verification Framework

AI findings are candidate suggestions until reviewed by an investigator. Every detection record maintains an explicit review status:

- **`UNREVIEWED` (Default):** AI candidate detection populated in timeline search gallery.
- **`VERIFIED`:** Investigator has manually inspected frame and confirmed target presence.
- **`REJECTED`:** Investigator has inspected frame and marked candidate detection as false positive.

---

## 4. API Response Schema (`GET /api/ai/detections`)

```json
{
  "total_candidate_detections": 12,
  "detections": [
    {
      "detection_id": "DET-2026-081",
      "artifact_id": "ART-2026-904",
      "model_name": "yolov8n.onnx",
      "model_version": "v8.0.200",
      "inference_engine": "ONNX Runtime CPU Provider",
      "target_class": "vehicle",
      "confidence_score": 0.885,
      "raw_frame_timestamp": "2026-08-25 14:20:02",
      "normalized_utc_timestamp": "2026-08-25 14:17:48Z",
      "bounding_box_pixels": [140, 95, 210, 160],
      "review_status": "VERIFIED",
      "forensic_disclaimer": "AI-Generated Analytical Finding; requires human investigator verification."
    }
  ]
}
```
