# Historical courses and exclusion rules

Read this file before selecting any major-elective or Free Elective course. A course's current `ME(...)` or `FE(...)` classification does not override the student's historical course status.

## Collect available history

Ask about history once, after the term, major, and admission year are settled.
Offer the standard progression as an option and accept free-text exceptions.
Continue with the available facts if the student does not supply history.
Treat the student's answer as settled for the conversation; do not ask again or mark every elective as uncertain.
Use these sources when available:

1. The student's statement that earlier terms followed the standard plan.
2. A list of completed course codes with their status.
3. A list of courses currently enrolled in or already selected for earlier terms.
4. An exported CourseDB Timetable JSON or the student's earlier semester plans.

Apply each status as follows:

- `COMPLETED`: the student confirms completion or the standard plan covers it; hard exclusion from the new plan.
- `ATTENDED`: treat as completed unless the student says it was failed, withdrawn, or dropped.
- `ENROLLED`: the student is taking it; exclude from the new plan unless the student explicitly asks about a retake or replacement.
- `PLANNED`: it appears in an earlier plan; exclude it from the new plan by default to avoid duplicating the earlier plan, but mark the exclusion as plan-based rather than completed evidence.
- `FAILED`, `WITHDRAWN`, or `DROPPED`: do not hard-exclude it; list it as a possible reattempt subject to the Handbook and the student's intent.
- `UNKNOWN`: do not silently treat it as completed. Ask about it only if it changes a selection; otherwise mention it once in the final caveats.

The standard plan establishes completion of earlier-term required courses only. It does not establish which electives the student took.
Do not read earlier Handbook terms only to rebuild history. If an elective candidate is a fixed required course from an earlier term that you already read, exclude it.

If the student provides only course names without exact codes, ask for the codes or clearly label the history as unverified. Do not infer equivalence from a similar course name alone.

## Parse a previous Timetable JSON

For a JSON object matching `references/timetable-json.md`, collect `entries[].courseCode`. The `entries` represent planned timetable entries, not proof of completion. Use the plan's `year` and `semester` to determine whether it precedes the target term. If a previous JSON entry has no `courseCode`, do not guess from `label`; ask for the exact code.

Build these sets from available history before querying elective candidates.
If history is missing, record it as unknown rather than assuming that the sets represent a complete record:

- `completedCourseCodes`: exact codes from confirmed `COMPLETED` history.
- `priorPlanCourseCodes`: exact codes from earlier `ENROLLED` or `PLANNED` plans.
- `reattemptableCourseCodes`: codes marked `FAILED`, `WITHDRAWN`, or `DROPPED`.
- `uncertainCourseCodes`: codes whose status or identity is unclear.

## Exclude before classification

Apply exclusions in this order:

1. Start with the Handbook's fixed anchors and the `ME(...)`/`FE(...)` candidate catalogues.
2. Remove every exact code in `completedCourseCodes` from every candidate pool, regardless of whether the same course is classified as a major required course, major elective, `FE(ALL)`, or `FE(<subjectCode>)`.
3. Remove `priorPlanCourseCodes` by default when the earlier plan is before the target term. Keep the reason as “already planned/enrolled earlier”, not “completed”.
4. Do not remove `reattemptableCourseCodes` automatically; show them separately and select them only when the student confirms a reattempt is intended.
5. Prefer candidates outside `uncertainCourseCodes`. If one is still selected, mention it once in the final caveats.

For example, if `COMP3173` was already a required course in an earlier term but also appears in an `FE(ALL)` catalogue, exclude `COMP3173` from the Free Elective candidates before judging whether it is a relevant or attractive elective. Course classification is not a substitute for course-history exclusion.

If history uses an original code and the current catalogue uses a known concrete variant, consult `references/course-code-expansion-map.md`. The map supplies observed lookup relationships, not proof of academic equivalence. Keep a related variant pending until the student or Handbook establishes how that history applies. Do not infer equivalence from a shared prefix, title, or an invented `800X` suffix.

## Missing history does not block previews

The current MCP tools read Handbook, course classifications, and Offerings only. They do not read an arbitrary student's completed-course record or personal Planner. A DEV API Key authenticates the developer integration; it does not identify the student whose plan is being generated. Do not invent a `get_previous_plan` tool or use a developer account's private Planner as the student's history.

If the student does not provide history and the target term has ME/FE choices:

- Query the Handbook, elective candidates, and their target-term Offerings.
- Select a provisional combination based on available facts and student preferences.
- Include selected electives in the JSON and temporary preview.
- State once, in the final caveats, that duplicate-course exclusions, requirement fulfillment, and academic eligibility remain unverified where evidence is missing.

Do not restrict the preview to fixed anchors because history is missing.
A preview does not require a confirmed academic plan.
If the student asks to ignore prerequisites, preserve that assumption and create the requested provisional preview.
Keep warnings outside the strict JSON.

If the student provides a prior plan but not completion status, treat it as `PLANNED`/`ENROLLED` evidence only. Never claim that those courses were completed.
