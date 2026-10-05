# Planning examples

These fixtures illustrate decisions with mock CourseDB results. Names, IDs, and values are examples, not live academic records.

## Ordinary selection and export

The student supplies a CST cohort, an explicit Fall term, and completed-course history. The Handbook has a fixed COMP2001 requirement of three units. Course details return:

```json
{
  "courses": [
    {
      "courseCode": "COMP2001",
      "recordStatus": "FOUND",
      "course": {
        "courseCode": "COMP2001",
        "courseName": { "en": "Example Course", "zh": "示例课程" },
        "units": 3,
        "department": "Computing",
        "faculty": "Science",
        "prerequisiteText": "COMP1001",
        "exclusionText": null,
        "description": "Example description",
        "classification": {
          "term": { "year": 2026, "semester": "FALL" },
          "offeringUnits": ["Science"],
          "offeringProgrammes": ["CST"],
          "curriculumTypes": ["MR"],
          "electiveTypes": [],
          "typeTokens": ["MR(CST)"]
        }
      }
    }
  ]
}
```

The batch Offering response contains:

```json
{
  "term": { "calendarYear": 2026, "calendarSeason": "FALL" },
  "courses": [
    {
      "courseCode": "COMP2001",
      "courseName": { "en": "Example Course", "zh": "示例课程" },
      "recordStatus": "FOUND",
      "sessionCount": 1,
      "sessionsTruncated": false,
      "sessions": [
        {
          "id": "4bec6c63-93c7-49bb-a0b5-d19a5c33bf63",
          "session": "S100",
          "schedule": "Mon 10:00-11:50",
          "offeringUnit": "Science",
          "offeringProgramme": "CST",
          "curriculumType": "MR",
          "electiveType": null,
          "typeTokens": ["MR(CST)"],
          "requirementsRaw": "School eligibility text",
          "remarksRaw": null,
          "rawLecturer": "Source teacher names",
          "lecturers": [],
          "hasAssignedLecturer": false,
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
          ]
        }
      ]
    }
  ]
}
```

Select S100 if it fits the supplied history and anchors. Use the three detail units and preserve the raw eligibility uncertainty. Export the matching entry in [timetable-json.md](timetable-json.md). The name comes from the course object and the ID comes from the session.

## Expansion and units

The Handbook lists WPEX2023 for three units. Its Offering is `NO_RECORD`. The observed map lists WPEX20238001 as a concrete variant.

Read the variant's Offering and details. If its current units are four, report three required units and four selected units separately. Do not substitute three units. Keep the Handbook mapping provisional unless its rules support that variant.

If the student previously completed WPEX2023, keep the variant's history status pending until equivalence is established. The map alone does not prove completion of WPEX20238001.

## Multiple lecturers

Suppose the selected session has these linked lecturer rows:

```json
[
  {
    "id": "row-1",
    "lecturerId": "teacher-1",
    "lecturerName": "Resolved Teacher",
    "rawLecturer": "Source spelling",
    "sequence": 1
  },
  {
    "id": "row-2",
    "lecturerId": null,
    "lecturerName": null,
    "rawLecturer": "Unresolved Teacher",
    "sequence": 2
  }
]
```

Display Resolved Teacher, then Unresolved Teacher. Preserve the second row's unresolved status. Do not add parent raw text as another teacher. If rows are absent, display parent raw text without inventing a resolved lecturer ID.

## Classification disagreement

The major catalogue lists a course through `ME(CST)`. Its latest classified-term details confirm that token. The target session contains only `FE(ALL)`.

Keep the course available as an Offering candidate, but leave major-elective coverage pending. Do not replace target tokens with the latest-term tokens. Apply the same pending status if target tokens are empty.

## Unknown or invalid time

A slot has `day: null`, `startMinutes: null`, and `endMinutes: null`, with raw time text. Preserve the nulls in JSON if other limits fit. Report unknown conflict status.

If `day` is `TBA` or the start minute is 1500, the slot does not fit the import schema. Keep the selected session outside the importable subset. Report the original values and pending export. Do not convert `TBA` to a guessed weekday or trim away the slot.

For complete Monday intervals 600–660 and 660–720, report no overlap. Intervals 600–660 and 650–720 overlap from 650 to 660.

## Project session

An FYP session is `FOUND`, has no slots, and has a null schedule. Preserve its real ID, use `timeSlots: []` and `schedule: ""`, and describe supervisor arrangements.

Do not mark it conflict-free. If explicit project slots exist, compare them normally. A project with `NO_RECORD` still needs the applicable code-map follow-up and stays outside the JSON.

## Complete pagination

A batch contains 100 sessions, `sessionCount: 101`, and `sessionsTruncated: true`. Its last session is `{session: "S100", id: "last-session-id"}`.

Call `list_course_offering_sessions` for the same course and term with that `after` value. If it returns S101 and `nextCursor: null`, merge by ID. A total of 101 unique IDs completes the observed enumeration.

If a request begins at the last record, an empty page can remain `FOUND`. It does not mean that the course has no Offering.

## Paging failure or repetition

Suppose a later page returns a tool error, repeats a cursor, or changes the full count from 101 to 102. Stop automatic paging and keep already-read sessions.

Report incomplete candidate comparison. Do not conclude that every session conflicts. If a usable known session is selected, label the plan provisional and disclose the missing alternatives.

If the final cursor is null but unique IDs do not match the full count, report the mismatch. Even matching counts do not guarantee an immutable snapshot during imports.
