---
name: plan-semester-courses
description: Plan a student's semester from CourseDB facts and course history. Compare requirements, elective choices, and timetable conflicts. Return a provisional recommendation and a temporary preview link. Use Timetable import JSON when preview creation fails.
metadata:
  version: "2.1"
---

# Plan Semester Courses

Use CourseDB facts to draft a provisional semester plan. Do not write to a Planner or claim official academic approval.

Read [mcp-tools.md](references/mcp-tools.md) before calling tools. Read [planning-rules.md](references/planning-rules.md) before selecting courses. Read [timetable-json.md](references/timetable-json.md) before exporting a plan.

Load these references when their conditions apply:

- If the student supplies a program name, read [program-code-map.md](references/program-code-map.md).
- If the plan includes GE, read [ge-programme.md](references/ge-programme.md). Preserve its levels, categories, and unit rules.
- If an Offering returns `NO_RECORD` or a code contains `800X`, read [course-code-expansion-map.md](references/course-code-expansion-map.md).
- Before selecting electives, read [history-and-exclusions.md](references/history-and-exclusions.md).
- For worked examples and failure cases, read [planning-examples.md](references/planning-examples.md).

## Establish context

Collect the program code, admission year, current study year, target Handbook year and term, target calendar term, and course history. Ask only for missing facts.

Use the canonical program code from the map. The Handbook lookup uses its stored code, not a course prefix.

Keep `studyYear` and `studyTermCode` separate from `calendarYear` and `calendarSeason`. A future calendar term does not establish the target Handbook year. If the timeline is ambiguous, ask the student.

Record history separately as completed, enrolled, planned, failed, withdrawn/dropped, or unknown. Earlier Timetable entries prove a plan, not completion. Exclude completed courses and earlier enrolled/planned courses before choosing electives. Keep uncertain status and requested retakes visible.

## Read requirements and candidates

1. Call `get_handbook_term_requirements` for the exact program, cohort, and Handbook term. If the tool fails, report the failure. Do not interpret an error as an empty Handbook or substitute another cohort.
2. Preserve each requirement's section, ID, type, `requiredUnits`, flexibility, pattern, notes, fixed course, and alternatives. Establish fixed `REQUIRED`/`CORE` anchors first.
3. Read relevant major-elective and exact `FE(...)` catalogues. Follow `hasMore` pages. Apply history exclusions before selecting candidates. Literal filters do not perform semantic search.
4. Build an exact candidate list. Use GE and observed code maps only when relevant. Query at most 50 codes per batch.
5. Call `get_course_details` for missing course facts and for selected codes whose current units or classification need confirmation. Use each concrete variant's own facts.
6. Call `get_course_offerings` for the explicit calendar term. If results are truncated, continue with `list_course_offering_sessions` as described in the tool reference.
7. For a Handbook-fixed code with `NO_RECORD`, query every recorded concrete variant before finalizing. Query variant details as well. If no mapping exists, report the unresolved code.

Do not invent suffixes, wildcard queries, or a natural-language search tool. For exploratory courses, expand only selected candidates or codes that the student explicitly asks to expand.

## Select sessions and explain evidence

Arrange anchors first, major electives second, Free Electives third, and remaining courses last. Select at most one session per course. Preserve usable alternatives when the choice depends on preferences.

Latest classified-term course metadata supplies candidate classification. Target-term session metadata supplies that session's classification. If they conflict or the target classification is missing, keep elective fulfillment pending.

Use current course-detail units for selected courses. Preserve differences from Handbook required units. A code mapping does not establish equal units or institutional equivalence.

Read names from the course-level `courseName`. Read linked lecturers in sequence order, with resolved names before raw names. Keep `requirementsRaw` and `remarksRaw` as source text.

Compare complete same-day time intervals. Adjacent intervals do not overlap. Missing or invalid times leave conflict status unknown. A recorded project session without fixed times can remain provisional.

Do not infer prerequisites, graduation eligibility, or cancellation from raw text, classification, or `NO_RECORD`. Separate retrieved facts from preference-based ranking and other reasoning.

## Follow-ups and final output

Keep the current context, requirements, history, details, session IDs, pagination completeness, and unresolved issues across follow-ups. Query only changed or new exact courses. If an anchor changes, recalculate affected conflicts.

Describe the planning context, recommended courses and sessions, requirement coverage, history exclusions, known conflicts, unknowns, and viable alternatives. Report required units separately from confirmed covered units. Do not double-count a course without explicit supporting rules.

If history is missing, keep electives as candidates. Export only verified fixed-anchor sessions and explain that elective selection remains pending.

Build strict Timetable JSON for the primary recommendation. Use real session IDs and the actual calendar term. Keep uncertainties and alternatives outside the JSON. Follow the import limits in the export reference.

If `create_timetable_preview` is available, generate a UUID for `requestId`. Call the tool with this UUID, a short name, and the Timetable object as `data`. Reuse the UUID and identical arguments for a retry. Return the preview link and expiry on success. Explain that anyone with the link can view the draft. A signed-in user can save a personal copy. Do not claim that the tool saves to an account.

If the tool is unavailable or fails, explain the failure and return the strict import JSON. Do not retry a quota error with another UUID or key.
