# 🔒 Flow 08: Cryptographic Verification & Zero-Transcoding Evidence Export

> **Module:** `backend/app/modules/export/`  
> **Status:** `✅ Completed (16/16 Flow 08 Tests Passing, 121/121 Overall)`  
> **Purpose:** Export forensic-grade time-bound video slices without quality loss (-c copy), generate HMAC-signed `.sync.json` sidecar manifests, package courtroom `.zip` evidence bundles, and perform multi-scenario anti-tampering verification and 1-click reverse-lookup manifest recovery.

---

## 🛡️ Anti-Tampering Verification Matrix

| Scenario | Trigger / Attack Vector | Verification Outcome |
| :--- | :--- | :--- |
| **Authentic Match** | Unmodified video + Original `.sync.json` | `VERIFIED_MATCH` (100% Authentic proof) |
| **Tampered Video** | Video altered, re-encoded, or spliced | `HASH_MISMATCH` (Alert: Content altered) |
| **Tampered Manifest** | Timestamp/camera ID altered for fake alibi | `METADATA_TAMPERED` (Alert: HMAC seal broken) |
| **Mismatched Pair** | Video A verified against Manifest B | `HASH_MISMATCH` (Alert: Belongs to different clip) |
| **Lost Manifest Recovery** | Only video file provided (lost `.sync.json`) | `VERIFIED_MATCH` (Recovered via Case database) |

---

## 📡 REST API Endpoints

| Endpoint | Method | Status Code | Purpose |
| :--- | :--- | :--- | :--- |
| `/api/v1/export/slice` | `POST` | `201 Created` | Slices time range, generates signed `.sync.json`, records in DB. |
| `/api/v1/export/{export_id}` | `GET` | `200 OK` | Retrieves export metadata and download links. |
| `/api/v1/export/download/{export_id}/video` | `GET` | `200 OK` | Direct download of zero-transcode `.mp4` video. |
| `/api/v1/export/download/{export_id}/manifest` | `GET` | `200 OK` | Direct download of signed `.sync.json` manifest. |
| `/api/v1/export/download/{export_id}/bundle` | `GET` | `200 OK` | Downloads `.zip` containing both `.mp4` and `.sync.json`. |
| `/api/v1/export/verify` | `POST` | `200 OK` | Cryptographic anti-tampering verification engine. |
| `/api/v1/export/recover-by-hash` | `POST` | `200 OK` | 1-Click recovery of lost `.sync.json` using video SHA-256. |

---

## 🧪 Test Verification (16 Flow 08 Tests / 121 Overall)

* `test_export_model.py`: Table creation, cascade relationships, enum validation, and manifest JSON serialization.
* `test_export_slicer.py`: Stream hashing, FFmpeg `-c copy` slicing, sector boundary interpolation.
* `test_export_service.py`: Service orchestration, ZIP bundling, HMAC signature verification, tamper detection, and reverse lookup.
* `test_export_api.py`: Full E2E REST API endpoints, binary video downloads, JSON manifest downloads, ZIP bundle downloads, and 400/404 error guards.
