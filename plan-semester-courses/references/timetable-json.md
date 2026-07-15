# Timetable import JSON

The final response must end with a strict JSON code block that can be imported by CourseDB Timetable. Read this file before producing the final plan.

## Required top-level shape

Use the actual Offering calendar term, not the Handbook study-term code:

```json
{
  "version": 2,
  "year": 2026,
  "semester": "FALL",
  "entries": [],
  "customBlocks": []
}
```

`semester` must be one of `SPRING`, `SUMMER`, `FALL`, or `WINTER`. `customBlocks` should be an empty array unless the student explicitly asks for personal blocks; the MCP tools do not provide personal blocks.

## Entry shape

Create one entry for each selected course/session in the recommended plan:

```json
{
  "id": "COMP2001-1001",
  "label": "COMP2001 1001",
  "courseCode": "COMP2001",
  "courseNameEn": "Example Course",
  "schedule": "Mon 10:00-11:50; Wed 10:00-11:50",
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
```

Rules:

- `id` must be a deterministic string unique within the JSON. Use `<courseCode>-<session>`, replacing whitespace in the session with `-` when needed. Do not use a random ID.
- `label` should be `<courseCode> <session>`.
- `courseNameEn` comes from the Offering course name when available. Omit it only when the name is unavailable.
- `schedule` must always be a string. Use the Offering `schedule`, or `""` when it is null or unavailable.
- Copy `timeSlots` from the selected Offering session. Preserve `sequence`, `day`, `startMinutes`, `endMinutes`, `rawTime`, `location`, and `locations` when present. A null day or time is valid uncertainty; do not invent a time.
- `colorIndex` is a finite number. Assign `0, 1, 2, ...` in the order of entries, wrapping after the available palette if necessary.
- Include only one session per selected course. Do not include alternative sessions in `entries`.
- Do not include a course with `NO_RECORD` in `entries`; report it separately as unresolved.
- A `FOUND` FYP/project course with no `timeSlots` may still be included with `timeSlots: []` and `schedule: ""`. Explain outside the JSON that its timetable is project/supervisor-arranged.

## Output discipline

The human-readable explanation may appear before the JSON, but the final section must be headed `Timetable JSON` and contain valid JSON only. Do not put Markdown comments, trailing commas, conflict annotations, requirement labels, or explanatory text inside the JSON object. Keep requirement categories, conflicts, unresolved Offering records, and alternatives in the preceding explanation.

The JSON represents the recommended plan only. If multiple plans are viable, provide separate human-readable alternatives and generate JSON for the primary recommendation unless the student asks for JSON for every alternative.
