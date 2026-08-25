# Feature 07: Evidence Search & Timeline Event Query Engine

**Back to [[MVP/MVP|MVP]]**

---

## 1. Primary Objective

The Evidence Search Engine provides a structured query interface over SQLite metadata tables (`sector_map`, `derived_artifacts`, `timeline_calibrations`, `ai_detections`).

Investigators query events using parameterized filters combining camera channels, raw/normalized timestamp windows, recovery statuses, and human-verified AI candidate tags.

---

## 2. Query Parameters & Filter Options

| Parameter Name | Data Type | Supported Filter Options | Description |
| :--- | :--- | :--- | :--- |
| `channel_id` | `INTEGER` | `1`, `2`, `3`, `4` | Target camera channel ID |
| `start_time_utc` | `DATETIME` | ISO-8601 UTC string | Start of search window |
| `end_time_utc` | `DATETIME` | ISO-8601 UTC string | End of search window |
| `recovery_status`| `TEXT` | `RECOVERED`, `PARTIAL`, `FRAGMENTED` | Stream recovery status filter |
| `ai_class` | `TEXT` | `person`, `vehicle` | Secondary AI detection class |
| `min_confidence` | `REAL` | `0.50` to `1.00` | Minimum AI model confidence score |
| `review_status` | `TEXT` | `VERIFIED`, `REJECTED`, `UNREVIEWED` | Human-in-the-Loop review status |

---

## 3. SQL Query Construction & Target Metrics

```sql
SELECT d.detection_id, a.artifact_id, a.channel_id,
       d.raw_frame_timestamp, d.normalized_utc_timestamp,
       d.target_class, d.confidence_score, d.review_status
FROM ai_detections d
JOIN derived_artifacts a ON d.artifact_id = a.artifact_id
WHERE a.channel_id = :channel_id
  AND d.target_class = :ai_class
  AND d.confidence_score >= :min_confidence
  AND d.review_status = :review_status
  AND d.normalized_utc_timestamp BETWEEN :start_time AND :end_time
ORDER BY d.normalized_utc_timestamp ASC;
```

- **Target Query Latency:** Sub-second response time on standard case databases (indexed SQLite schema).
- **Target Result Capacity:** Paginated responses (e.g., 50 records per API payload).

---

## 4. API Specification (`GET /api/evidence/search`)

```json
{
  "total_matches": 8,
  "page": 1,
  "page_size": 50,
  "results": [
    {
      "detection_id": "DET-2026-081",
      "artifact_id": "ART-2026-904",
      "channel_id": 2,
      "raw_timestamp": "2026-08-25 14:20:02",
      "normalized_utc": "2026-08-25 14:17:48Z",
      "target_class": "person",
      "confidence_score": 0.885,
      "review_status": "VERIFIED",
      "artifact_sha256": "4f8b91a0c..."
    }
  ]
}
```
