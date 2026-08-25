# Feature 05: Raw Timestamp Preservation & UTC Timeline Normalization

**Back to [[MVP/MVP|MVP]]**

---

## 1. Primary Timestamp Principles

In digital forensics, **original raw in-stream timestamps must never be overwritten, modified, or deleted**.

Internal DVR clocks suffer from numerous anomalies:
- DVR clock set incorrectly or failing RTC battery.
- Desynchronization across heterogeneous camera channels.
- Timezone misconfigurations and Daylight Saving Time (DST) jumps.
- Lack of Network Time Protocol (NTP) synchronization.

Locus implements a **two-layer timeline architecture**:
1. **Raw Metadata Layer:** Preserves exact, unmodified in-stream timestamp bytes extracted from sector headers.
2. **Normalized Analytical Layer:** Applies calculated clock offsets and timezone interpretations to construct a synchronized master UTC timeline.

```text
Raw In-Stream Timestamp ──► Timezone Interpretation ──► Clock Drift Offset ──► Normalized UTC Timeline
   (Unmodified Source)                                                           (Analytical Overlay)
```

---

## 2. Multi-Camera Synchronization Model

To synchronize multi-camera grid playback without modifying source evidence:

```
Camera Channel 1 (Raw: 14:00:00) ──► Offset: -00:02:14 ──┐
                                                         ▼
Camera Channel 2 (Raw: 14:05:10) ──► Offset: -00:07:24 ──┼─► Master Synchronized Timeline (13:57:46 UTC)
                                                         ▲
Camera Channel 3 (Raw: 13:50:00) ──► Offset: +00:07:46 ──┘
```

---

## 3. Timeline Calibration Data Schema (`timeline_calibrations`)

| Column Name | Data Type | Sample Value | Description |
| :--- | :--- | :--- | :--- |
| `id` | `INTEGER` (PK) | `301` | Calibration record ID |
| `channel_id` | `INTEGER` | `2` | Camera channel number |
| `raw_timestamp` | `TEXT` | `"2026-08-25 14:05:10"` | Original raw timestamp string |
| `offset_seconds` | `INTEGER` | `-444` | Calculated clock drift offset |
| `timezone_offset` | `TEXT` | `"+05:30"` | Timezone string |
| `normalized_utc` | `TEXT` | `"2026-08-25 08:27:46Z"` | Calculated UTC timestamp |
| `calibration_method`| `TEXT` | `"OSD_VISUAL_ANCHOR"` | `OSD_VISUAL_ANCHOR`, `CAD_INCIDENT`, `NTP_BASELINE` |
| `confidence` | `REAL` | `0.95` | Offset estimation confidence score |
