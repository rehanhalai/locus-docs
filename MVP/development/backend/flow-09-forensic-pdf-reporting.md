# 📑 Flow 09: Forensic PDF Case Dossier & Chain-of-Custody Report Generator

> **Module:** `backend/app/modules/reports/`  
> **Status:** `✅ Completed (6/6 Flow 09 Tests Passing, 131/131 Overall)`  
> **Purpose:** Generate court-admissible, tamper-evident official PDF case reports and certificates of authenticity. Summarizes case metadata, hardware evidence baseline hashes (SHA-256 / MD5), DVR file systems, multi-camera timeline calibration offsets, AI detected events index, zero-transcode export seals, and the immutable chain-of-custody audit log.

---

## 🏛️ Courtroom Sections Included in the PDF Report

1. **Executive Case Header:** Case number, case name, investigator, date ingested, status, description.
2. **Physical Evidence Hardware & Hash Baselines:** Source devices, partition types, detected DVR brand/file system (DHFS/HKFS/WFS), and baseline SHA-256 & MD5 hashes.
3. **Multi-Camera Timeline Calibration:** Camera IDs, calibrated time offsets ($\Delta t$), reasons, and calibration officer.
4. **AI Video Analytics & Chronological Event Index:** Calibrated timestamps, cameras, detected object classes (`PERSON`, `CAR`, `TRUCK`, `KNIFE`, `BACKPACK`, `MOTION`), confidence %, and bounding boxes.
5. **Cryptographic Evidence Exports & Manifest Seals:** Export IDs, filenames, sector ranges, exported file SHA-256 hashes, and HMAC sidecar seals.
6. **Immutable Chain-of-Custody Audit Trail:** Full chronological log of actions (`CASE_INGESTION`, `DEVICE_IDENTIFIED`, `SECTOR_INDEXED`, `AI_ANALYTICS_COMPLETED`, `EVIDENCE_EXPORTED`, `EXPORT_VERIFIED`, `REPORT_GENERATED`).
7. **Legal Forensic Certificate:** Section 65B & Federal Rule 902 certification seal.

---

## 📡 REST API Endpoints

| Endpoint | Method | Status Code | Purpose |
| :--- | :--- | :--- | :--- |
| `/api/v1/reports/pdf/{case_id}` | `GET` | `200 OK` | Direct binary download/stream of the official courtroom `.pdf` report. |
| `/api/v1/reports/generate/{case_id}` | `POST` | `201 Created` | Generates report and returns report metadata with download URL. |
| `/api/v1/reports/summary/{case_id}` | `GET` | `200 OK` | Retrieves aggregate statistical summary for the case. |

---

## 🧪 Test Verification (6 Flow 09 Tests / 131 Overall)

* `test_generate_pdf_dossier_direct_generator`: Validates PDF generation with valid `%PDF-` header and all 6 report sections populated.
* `test_download_case_pdf_report_api_and_audit`: Verifies API endpoint streaming, content type `application/pdf`, and automatic creation of `REPORT_GENERATED` audit log.
* `test_generate_case_report_post_api`: Verifies POST endpoint returning 201 Created and metadata.
* `test_get_case_summary_metadata_api`: Verifies statistical count aggregation.
* `test_download_pdf_missing_case_returns_404`: 404 security guard.
* `test_get_summary_missing_case_returns_404`: 404 security guard.
