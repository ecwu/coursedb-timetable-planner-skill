# Historical courses and exclusion rules

Read this file before selecting any major-elective or Free Elective course. A course's current `ME(...)` or `FE(...)` classification does not override the student's historical course status.

## Collect the student's history first

When the target Handbook term contains an elective slot, ask for at least one of these sources before choosing elective courses:

1. A list of completed or attended course codes.
2. A list of courses currently enrolled in or already selected for earlier terms.
3. An exported CourseDB Timetable JSON or the student's earlier semester plans.

For each course, ask or infer a status and keep the status visible:

- `COMPLETED` or `ATTENDED`: the student has taken the course; hard exclusion from the new plan.
- `ENROLLED`: the student is taking it; exclude from the new plan unless the student explicitly asks about a retake or replacement.
- `PLANNED`: it appears in an earlier plan; exclude it from the new plan by default to avoid duplicating the earlier plan, but mark the exclusion as plan-based rather than completed evidence.
- `FAILED`, `WITHDRAWN`, or `DROPPED`: do not hard-exclude it; list it as a possible reattempt subject to the Handbook and the student's intent.
- `UNKNOWN`: do not silently treat it as completed or available. Ask the student to clarify before using it as an elective.

If the student provides only course names without exact codes, ask for the codes or clearly label the history as unverified. Do not infer equivalence from a similar course name alone.

## Parse a previous Timetable JSON

For a JSON object matching `references/timetable-json.md`, collect `entries[].courseCode`. The `entries` represent planned timetable entries, not proof of completion. Use the plan's `year` and `semester` to determine whether it precedes the target term. If a previous JSON entry has no `courseCode`, do not guess from `label`; ask for the exact code.

Build these sets before querying elective candidates:

- `completedCourseCodes`: exact codes from `COMPLETED` or `ATTENDED` history.
- `priorPlanCourseCodes`: exact codes from earlier `ENROLLED` or `PLANNED` plans.
- `reattemptableCourseCodes`: codes marked `FAILED`, `WITHDRAWN`, or `DROPPED`.
- `uncertainCourseCodes`: codes whose status or identity is unclear.

## Exclude before classification

Apply exclusions in this order:

1. Start with the Handbook's fixed anchors and the `ME(...)`/`FE(...)` candidate catalogues.
2. Remove every exact code in `completedCourseCodes` from every candidate pool, regardless of whether the same course is classified as a major required course, major elective, `FE(ALL)`, or `FE(<subjectCode>)`.
3. Remove `priorPlanCourseCodes` by default when the earlier plan is before the target term. Keep the reason as “already planned/enrolled earlier”, not “completed”.
4. Do not remove `reattemptableCourseCodes` automatically; show them separately and select them only when the student confirms a reattempt is intended.
5. Do not select `uncertainCourseCodes` until the student clarifies them.

For example, if `COMP3173` was already a required course in an earlier term but also appears in an `FE(ALL)` catalogue, exclude `COMP3173` from the Free Elective candidates before judging whether it is a relevant or attractive elective. Course classification is not a substitute for course-history exclusion.

If history uses an original code and the current catalogue uses a known concrete variant, consult `references/course-code-expansion-map.md`. Exclude only the exact original/variant relationships recorded there; do not infer equivalence from a shared prefix, title, or an invented `800X` suffix.

## Missing history is a planning limitation

The current MCP tools read Handbook, course classifications, and Offerings only. They do not read an arbitrary student's completed-course record or personal Planner. A DEV API Key authenticates the developer integration; it does not identify the student whose plan is being generated. Do not invent a `get_previous_plan` tool or use a developer account's private Planner as the student's history.

If the student does not provide history and the target term has ME/FE choices:

- You may query the Handbook and fixed anchor Offerings.
- You may show elective candidates and their Offerings as an unfiltered candidate pool.
- Do not present an elective as a final recommendation or generate a fully confirmed timetable JSON. If the student asks for JSON anyway, include only verified fixed-anchor sessions and mark the JSON plan as incomplete outside the JSON block.
- State that the plan is provisional because historical course completion has not been verified.

If the student provides a prior plan but not completion status, treat it as `PLANNED`/`ENROLLED` evidence only. Never claim that those courses were completed.
