# Feature 09: Forensic Evidence Report Generator

**Back to [[MVP/MVP|MVP]]**

---

## 1. Objective & Forensic Scope

Module 10 (`Forensic Evidence Report Generator`) compiles all case data collected by Modules 01–09 into a single, structured, human-readable Forensic Evidence Report exported as both HTML and PDF.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    FORENSIC EVIDENCE REPORT COMPONENTS                      │
│                                                                             │
│  [CASE DETAILS]         Case ID, Investigator, Registration Timestamp.      │
│  [BASELINE HASHES]      Source image SHA-256 & MD5 at registration.         │
│  [HASH PARITY TABLE]    Derived artifact hashes vs. re-verified source.     │
│  [ARTIFACT PROVENANCE]  Byte offset, sector length, adapter version.        │
│  [AI FINDINGS SUMMARY]  Human-reviewed AI detection results only.           │
│  [INVESTIGATOR LOG]     All manual review actions with ISO-8601 timestamps. │
│  [DISCLAIMER SECTION]   Explicit uncertainty and scope limitations.         │
└─────────────────────────────────────────────────────────────────────────────┘
```

> [!IMPORTANT]
> **Forensic Scope Boundary:**
> The report generator is a structured export of data already recorded in SQLite.
> It does **not** perform new analysis, re-hash raw evidence, or generate derived
> media during export. All data in the report must be traceable to prior
> pipeline operations with audit log entries.

---

## 2. Requirements Addressed

| Requirement ID | Requirement Description |
| :--- | :--- |
| **REQ-M10-1** | Export structured HTML and PDF Forensic Evidence Reports. |
| **REQ-M10-2** | Include evidence baseline hashes, artifact hash parity tables, provenance details, human-reviewed AI findings, and uncertainty disclaimers. |

*Source: `PS/requirements-and-modules.md`, Module 10.*

---

## 3. PDF/HTML Generation Library

Locus uses **WeasyPrint** (Python library) to generate PDF output from Jinja2-rendered HTML templates:

- **Rendering pipeline:** FastAPI backend → Jinja2 HTML template → WeasyPrint → PDF binary → file export.
- **Why WeasyPrint:** Pure-Python, local execution (no headless browser required), deterministic page layout, and CSS-driven styling suitable for structured forensic table rendering.
- **Limitation:** WeasyPrint does not execute JavaScript; report templates must be static HTML/CSS only.

*See also: Technology Justification tables in `architecture-and-tech-stack.md` Section 2 and `project-overview.md` Section 7.*

---

## 4. SQLite Data Schema (Report Source Tables)

The report generator queries the following SQLite tables from `case_meta.db`:

| Table | Key Columns Used in Report | Report Section |
| :--- | :--- | :--- |
| `cases` | `case_id`, `case_name`, `investigator_id`, `created_at` | Case Details |
| `evidence_sources` | `evidence_id`, `file_name`, `file_size_bytes`, `baseline_sha256`, `baseline_md5`, `ingested_at` | Baseline Hashes |
| `derived_artifacts` | `artifact_id`, `file_name`, `artifact_sha256`, `artifact_md5`, `source_evidence_id`, `pre_export_verified` | Hash Parity Table |
| `provenance` | `artifact_id`, `source_offset_bytes`, `source_length_bytes`, `adapter_name`, `adapter_version`, `recovery_status` | Artifact Provenance |
| `ai_detections` | `artifact_id`, `model_name`, `target_class`, `confidence_score`, `frame_timestamp`, `review_status` | AI Findings Summary |
| `audit_log` | `event_timestamp`, `event_type`, `operator_id`, `description` | Investigator Review Log |

> [!WARNING]
> **Pre-export hash verification is mandatory:**
> Before compiling the report, the backend re-reads and re-hashes each source evidence
> image referenced. If the computed hash does not match `baseline_sha256`, the report
> export is **blocked** and an error is written to `audit_log` with type
> `HASH_MISMATCH_ABORT`. This prevents silent evidence corruption from appearing in
> an exported report.

---

## 5. API Endpoint

### `POST /api/v1/report/generate`

Triggers report generation for a given case. Queries all relevant SQLite tables,
renders the Jinja2 HTML template, converts to PDF via WeasyPrint, and saves both
formats to the case workspace directory.

**Request:**

```json
{
  "case_id": "CASE-2026-882",
  "investigator_id": "officer_smith_409",
  "export_formats": ["html", "pdf"],
  "include_ai_findings": true,
  "include_audit_log": true
}
```

**Response (success):**

```json
{
  "status": "success",
  "case_id": "CASE-2026-882",
  "report_id": "RPT-2026-001",
  "generated_at_utc": "2026-08-25T14:30:00Z",
  "source_verification": {
    "evidence_id": "EVD-2026-001",
    "file_name": "dahua_nvr_dump.dd",
    "baseline_sha256": "8c2a5f4b9d1e38a7c6e0f2b4d6a8c1e3f5b7d9a0c2e4f6a8b1d3f5a7c9e1b3d5",
    "baseline_md5": "c4f8a1e2d3b5c7a9e0f1b3d5a7c9e1b3",
    "pre_export_verification": "VERIFIED_MATCH"
  },
  "artifacts_included": 3,
  "ai_findings_included": 7,
  "audit_events_included": 24,
  "export_paths": {
    "html": "./workspace/cases/CASE-2026-882/reports/RPT-2026-001.html",
    "pdf": "./workspace/cases/CASE-2026-882/reports/RPT-2026-001.pdf"
  }
}
```

**Response (hash mismatch — export blocked):**

```json
{
  "status": "error",
  "error_code": "HASH_MISMATCH_ABORT",
  "evidence_id": "EVD-2026-001",
  "detail": "Pre-export hash verification failed. Source image hash does not match registered baseline. Report export blocked. Audit log entry written.",
  "audit_log_event_id": "AUD-2026-00419"
}
```

---

## 6. Report Structure (Rendered Output)

The generated report contains the following sections in order:

1. **Cover Page:** Case ID, investigator name, generation timestamp (UTC), Locus version.
2. **Disclaimer:** Scope limitations, AI probabilistic label, non-legal-admissibility statement.
3. **Case Details:** Registration date, operator ID, evidence file metadata.
4. **Baseline Evidence Hashes:** SHA-256 and MD5 of each registered source image.
5. **Hash Parity Table:** Side-by-side derived artifact SHA-256/MD5 vs. source verification status.
6. **Artifact Provenance Table:** Byte offset, sector length, adapter name/version, and recovery status for each derived clip.
7. **AI Analytical Findings Summary:** Human-reviewed detections only (`Verified` status). Includes model name, confidence score, frame timestamp, and bounding box coordinates. `Rejected` and `Unreviewed` detections are excluded from the summary section but listed in an appendix.
8. **Investigator Review Log:** Chronological audit events (ISO-8601 UTC) for all analysis operations and review actions.
9. **Appendix:** Full AI detection log (all statuses), full audit log excerpt.

> [!NOTE]
> **Hash examples in this document (illustration only):**
> - Source image SHA-256 (64 hex chars): `8c2a5f4b9d1e38a7c6e0f2b4d6a8c1e3f5b7d9a0c2e4f6a8b1d3f5a7c9e1b3d5`
> - Derived artifact SHA-256 (64 hex chars): `4f8b91a0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a08d9e2f4a6b8c0d2e4`
> - MD5 (32 hex chars): `c4f8a1e2d3b5c7a9e0f1b3d5a7c9e1b3`

---

## 7. Forensic Safeguards & Limitations

| Safeguard | Description |
| :--- | :--- |
| **No new analysis during export** | Report compiles existing SQLite records only; no re-parsing of disk images. |
| **Pre-export hash gate** | Source image hash must match baseline before any report is written. |
| **Unreviewed AI not in summary** | Only `Verified` AI detections appear in the main findings; `Rejected`/`Unreviewed` are appendix-only. |
| **Explicit uncertainty sections** | Every report includes a disclaimer section with scope limitations and non-admissibility statement. |
| **Deterministic output** | Identical SQLite records + identical Jinja2 template version → identical report content. |
| **Local-only export** | Report files are written to the case workspace directory; no network transmission. |
