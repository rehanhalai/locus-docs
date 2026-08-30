# ⏱️ Flow 05: Multi-Camera Master Timeline Synchronization & Clock Calibration

> **Module:** `backend/app/modules/timeline/`  
> **Status:** `✅ Completed (9/9 Flow 05 Tests Passing, 90/90 Overall)`  
> **Purpose:** Synchronize multi-channel CCTV camera recordings onto a unified master timeline grid. Provides a non-destructive clock calibration layer ($\pm$ seconds/milliseconds) to correct DVR clock skew without altering raw video files or invalidating cryptographic hashes.

---

## 🎯 The Forensic Problem: Clock Skew & Drift

In multi-camera surveillance footage:
* DVR real-time clock (RTC) batteries degrade or lack Network Time Protocol (NTP) synchronization.
* **Camera 1 clock:** `10:14:00 AM` *(4 minutes fast)*
* **Camera 2 clock:** `10:10:00 AM` *(Real ground-truth time)*
* **Camera 3 clock:** `09:55:00 AM` *(15 minutes slow)*

Playing these streams simultaneously causes chronological paradoxes (e.g. suspect appears in Camera 2 before Camera 1).

### 🛡️ The Forensic Solution: Non-Destructive In-Memory Offsets

Modifying or re-encoding raw `.mp4` video files to change timestamps violates digital forensics chain-of-custody by altering SHA-256 hashes. Locus stores mathematical offsets in the `timeline_calibrations` table:

$$\text{Master Calibrated Timestamp} = \text{Raw Camera Timestamp} + \text{Offset Seconds}$$

---

## 🗄️ Database Schema: `timeline_calibrations` Table

```sql
CREATE TABLE timeline_calibrations (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    evidence_id VARCHAR(64) NOT NULL REFERENCES evidence_files(id),
    camera_id INTEGER NOT NULL,
    offset_seconds FLOAT NOT NULL DEFAULT 0.0,   -- e.g. +240.0 or -15.5
    reason VARCHAR(255),                         -- e.g. "Synced with NIST atomic clock"
    calibrated_by VARCHAR(128) NOT NULL,         -- Detective Vance
    updated_at DATETIME NOT NULL
);
```

---

## 📡 REST Endpoints

| Endpoint | Method | Status Code | Description |
| :--- | :--- | :--- | :--- |
| `/api/v1/timeline/{evidence_id}` | `GET` | `200 OK` | Retrieves unified multi-track timeline with calibrated start/end bounds for all camera channels. |
| `/api/v1/timeline/calibrate` | `POST` | `200 OK` | Sets or updates a camera's calibration offset and logs to `AuditLog`. |
| `/api/v1/timeline/calibrations/{evidence_id}` | `GET` | `200 OK` | Lists all active calibration offsets for an evidence file. |
| `/api/v1/timeline/calibrate/{evidence_id}/{camera_id}` | `DELETE` | `200 OK` | Resets a camera's calibration offset back to 0.0s and logs to `AuditLog`. |
| `/api/v1/timeline/sync-frame/{evidence_id}?timestamp=...` | `GET` | `200 OK` | Instantaneous multi-camera playback matrix resolving exact clip IDs and seek offsets (in seconds) for HTML5 video player grid tiles. |

---

## 🧪 Test Verification (9 Flow 05 Tests / 90 Overall)

Verified in `backend/tests/test_timeline.py` and `backend/tests/test_timeline_api.py`:

1. `test_timeline_calibration_math_and_audit` $\rightarrow$ Persists positive/negative offsets and records forensic `AuditLog` entry.
2. `test_timeline_delete_calibration` $\rightarrow$ Resets offset to 0.0s and logs reset action.
3. `test_master_timeline_empty_clips` $\rightarrow$ Handles evidence with no carved clips cleanly.
4. `test_set_and_get_camera_calibration_api` $\rightarrow$ Sets and queries calibration offsets via REST.
5. `test_master_timeline_synchronization_api` $\rightarrow$ Validates unified master bounding box and synchronized camera tracks.
6. `test_grid_sync_frame_resolver_api` $\rightarrow$ Resolves exact seek offset (e.g. 150.0s) for active camera grid tiles at master playhead position.
7. `test_grid_sync_frame_inactive_out_of_bounds` $\rightarrow$ Returns `is_active=False` when timestamp falls outside recorded bounds.
8. `test_reset_calibration_api` $\rightarrow$ Resets calibration via DELETE endpoint.
9. `test_timeline_missing_evidence_returns_404` $\rightarrow$ 404 security guard for invalid evidence ID.
