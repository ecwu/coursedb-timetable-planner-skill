# Timetable import JSON

End the planning response with strict JSON for the primary recommendation. Keep coverage, conflicts, alternatives, and unresolved records outside the JSON.

## Calendar term and entries

Use the actual Offering term. Keep `customBlocks` empty unless the student supplies personal blocks.

```json
{
  "version": 2,
  "year": 2026,
  "semester": "FALL",
  "entries": [
    {
      "id": "4bec6c63-93c7-49bb-a0b5-d19a5c33bf63",
      "label": "COMP2001 S100 - Example Course",
      "courseCode": "COMP2001",
      "courseNameEn": "Example Course",
      "schedule": "Mon 10:00-11:50",
      "timeSlots": [
        {
          "sequence": 1,
          "day": "Mon",
          "startMinutes": 600,
          "endMinutes": 710,
          "rawTime": "10:00-11:50",
          "location": "WLB 101",
          "locations": ["WLB 101"]
        }
      ],
      "colorIndex": 0
    }
  ],
  "customBlocks": []
}
```

The example ID represents a selected session. In a real plan, copy the actual returned `sessions[].id`. Do not copy the example ID or generate a synthetic `<courseCode>-<session>` ID.

## Field mapping

- `id`: selected session ID. Keep each ID unique and select only one session per course.
- `label`: course code, session, and English name when available. Chinese names can appear in the explanation or label.
- `courseCode`: exact concrete code, including any observed variant suffix.
- `courseNameEn`: course-level Offering `courseName.en`, then current course details, then Handbook/catalogue. Omit it only if all names are unavailable.
- `schedule`: copy the session string. Use `""` for null. Do not replace raw schedule text with an invented schedule.
- `timeSlots`: copy `sequence`, `day`, minute values, raw time, and location fields from the selected session.
- `colorIndex`: use integer palette indices in entry order. CourseDB currently has eight colors, so wrap indices modulo 8.

The import has no `courseNameZh`, lecturer, classification, units, requirement, or warning fields. Keep these facts in the explanation.

## Import limits

The current import schema accepts these limits:

- `year`: integer from 2000 to 2100. MCP reads remain limited to 2099.
- `semester`: `SPRING`, `SUMMER`, `FALL`, or `WINTER`.
- `entries` and `customBlocks`: at most 200 each.
- Entry `id`: at most 100 characters. `label` and `courseNameEn`: at most 255. `courseCode`: at most 32.
- Entry `schedule`: at most 2000 characters. `colorIndex`: integer from 0 to 100.
- Entry `timeSlots`: at most 50. Slot `sequence`: integer from 0 to 1000.
- Slot `day`: `Mon`, `Tue`, `Wed`, `Thu`, `Fri`, `Sat`, `Sun`, or null.
- Slot `startMinutes`: integer from 0 to 1439 or null. `endMinutes`: integer from 1 to 1440 or null.
- Slot `rawTime` and `location`: at most 255 characters. `locations`: null or at most 20 strings, each at most 255 characters.
- Optional entry location: at most 255 characters. Serialized data: at most 1,000,000 characters under the server schema.

For conflict analysis, a complete interval also requires `endMinutes > startMinutes`. Do not call an invalid interval usable merely because individual numbers fit the schema.

If an unknown day or out-of-range value prevents faithful export, keep that session outside the importable subset. Report it as pending. Do not guess weekdays, discard slots, truncate text, or replace invalid source values to force import.

Null day and minute values are valid uncertainty. Preserve them when they satisfy the schema. Explain that an importable record can still have unknown conflict status.

Custom blocks require a valid day, start and end minutes, integer color index, and bounded ID and label. Apply the same interval and text rules.

## Incomplete and project plans

Do not export `NO_RECORD` courses or alternative sessions. If a recommended session cannot fit the schema, export the remaining verified subset and identify the omitted course.

A `FOUND` project session with no fixed slots can use `timeSlots: []`. Preserve its schedule if present. For a null schedule, use `""`. Explain project/supervisor arrangements outside the JSON.

If student history is missing, export verified fixed-anchor sessions only. If paging is incomplete, identify the export as provisional. Do not claim that all alternatives were evaluated.

For multiple plans, export the primary recommendation unless the student requests each plan. Use valid JSON without comments or trailing commas.
