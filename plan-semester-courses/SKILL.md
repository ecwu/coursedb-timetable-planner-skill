---
name: plan-semester-courses
description: Plan one student's semester courses with the CourseDB read-only MCP tools, including Handbook requirements, major-elective candidates, Offering sessions, timetable conflicts, and follow-up checks for exact course codes. Use when a student asks what to take for a semester, wants to use a Handbook, needs major elective choices, or wants to compare a new course against an existing plan. Ask for the student's program, admission year, and current study year before planning.
compatibility: Requires an MCP client connected to the CourseDB /api/mcp endpoint and a valid DEV API Key.
metadata:
  version: "1.0"
---

# Plan Semester Courses

Use this skill to turn a student's Handbook requirements and course Offering data into a clearly explained, provisional semester plan. The MCP server provides facts; the skill performs the ordering, filtering, schedule comparison, and explanation. Do not write to a Planner or claim official academic approval.

Read [references/mcp-tools.md](references/mcp-tools.md) before making tool calls. Read [references/program-code-map.md](references/program-code-map.md) when the student gives a program name instead of a code or asks about abbreviations. Read [references/ge-programme.md](references/ge-programme.md) when the Handbook contains a GE requirement or the student asks about GE courses. Read [references/course-code-expansion-map.md](references/course-code-expansion-map.md) when an Offering lookup returns `NO_RECORD`, the student provides an `800X`-style code, or a course code may need a known concrete variant. Read [references/planning-rules.md](references/planning-rules.md) before selecting courses.

## Keep planning state across follow-ups

Maintain the current planning context during the conversation:

- Program code and name.
- Admission/cohort year.
- Current study year.
- Target Handbook study year and `studyTermCode`.
- Actual Offering calendar year and season.
- Selected or provisional anchor sessions.
- Elective candidates already checked and unresolved requirements.

When the student asks a follow-up about a course, session, or conflict, reuse this context. Do not repeat the Handbook and full elective-pool queries unless the student changes the cohort, target term, program, or planning assumptions. Query only the new exact course codes needed for the follow-up.

## Start the conversation

Ask for these facts before calling the Handbook tool:

- Program or major, preferably the exact program code.
- Admission/cohort year.
- Current study year, such as year 1, 2, 3, or 4.
- Target calendar term for the Offering lookup, such as `2026 Fall`.

Keep the Handbook term and actual calendar Offering term separate:

- `studyYear` plus `studyTermCode` identifies the position in the Handbook.
- `calendarYear` plus `calendarSeason` identifies when classes are actually offered.

Map the explicit calendar season to the Handbook term code using the reference table. `studyTermCode=1` means the first Handbook term and commonly corresponds to Fall/Sem 1; it does not by itself determine the student's year level. Do not automatically use the student's current study year as the target Handbook `studyYear` when the target term is in the future. Derive it only when the cohort timeline is unambiguous; otherwise ask the student to confirm the target Handbook year.

If the student says only “next semester”, ask for the calendar year and season or state the assumption explicitly before querying Offerings. If the student provides both a relative phrase and an explicit term, use the explicit term and call out any mismatch.

## Resolve the program code

Use the program-code map when the student provides a full name. Treat the code as exact and case-insensitive. In the current data model the same code is used to locate the Handbook and to match a course type such as `ME(CST)`.

Do not guess when multiple program codes could match a name. Ask the student to choose the intended code. Do not invent a code from a course prefix.

## Use the MCP tools in this order

1. Call `get_handbook_term_requirements` with the exact `programCode`, the admission year as `cohortYear`, and the requested `studyYear` and `studyTermCode`.

   If no active Handbook is returned, stop and ask for a corrected program code, admission year, or Handbook term. Do not substitute a neighboring cohort without telling the student.

2. Classify the returned requirements into planning buckets while preserving the source section, requirement ID, requirement type, units, notes, and course pattern:

   - Anchor courses: fixed `REQUIRED` or `CORE` requirements with a concrete course.
   - Major electives: an unfilled `ELECTIVE` requirement or courses listed by `ME(<programCode>)`.
   - GE requirements: preserve the exact GE level and category, such as `History and Civilization`, `Science, Technology and Society`, or `Experiential Learning`.
   - Other courses: remaining Handbook requirements or courses the student explicitly asks to consider outside the major-elective list.

3. Call `list_major_elective_courses` with the resolved program code only when the Handbook has a major-elective/elective slot that may use this pool, or when the student explicitly asks for major-elective candidates. Read every page when the result has `hasMore: true`. Use `courseCodePrefix` or `nameContains` only as literal database filters. Do not describe this as semantic search.

4. Build an exact candidate course-code list. Start with anchor courses, then add only the major-elective candidates relevant to the unresolved requirement, then add other courses. Do not make a small arbitrary “probe” call followed by a full call when the candidate pool is already known; batch the exact codes in one Offering request when there are at most 50. If more than 50 exact codes must be checked, split into chunks of 50.

5. Call `get_course_offerings` for the candidate course codes and the explicit calendar term. Send at most 50 course codes per call and split into multiple calls when necessary. Query every candidate when the student asks which candidates are offered; otherwise filter to a small, explainable shortlist before calling.
6. After every Offering response, collect all `NO_RECORD` results. For a concrete course listed in the Handbook as `REQUIRED`, `CORE`, or a fixed alternative, this is a blocking follow-up: consult the expansion map and query every mapped concrete variant before writing the plan. Do not treat the original `NO_RECORD` as final until this check is complete.

7. Choose at most one session for each selected course. Prefer sessions with complete time and location data. Compare sessions for same-day overlaps before finalizing the plan.

8. Present the plan in the required order: anchor major courses first, major electives second, and other courses last. Explain rejected candidates and unresolved data briefly.

Never finalize a plan while a Handbook-fixed course has an unchecked `NO_RECORD` expansion. For example, `CHI1103` with no direct Offering requires a follow-up Offering query for the mapped `CHI11038002` before reporting that the course is unavailable.

## Handle General Education requirements

When the Handbook or student request involves GE, read [references/ge-programme.md](references/ge-programme.md) and apply the official category structure:

1. Map each requirement to its exact GE level and category. Do not collapse all GE requirements into one generic elective pool.
2. For Level 1, fill one 3-unit course in each of Values and the Meaning of Life, Quantitative Reasoning, and History and Civilization.
3. For Level 2, fill two 3-unit courses under one theme or split across two themes: Culture, Creativity and Innovation; Science, Technology and Society; and Sustainable Communities.
4. For Level 3, fill one 3-unit course under any one capstone category: Service-Learning, Service Leadership Education, Experiential Learning, or Interdisciplinary Independent Study.
5. Use the GE reference table to build exact candidate codes. Filter by the student's stated interests only as an Agent interpretation, then query Offering data for the target calendar term. Do not query the entire GE catalogue by default.
6. If a concrete GE course is explicitly listed by the Handbook or has been selected as a candidate, and its base code has `NO_RECORD`, consult the course-code expansion map and query only known concrete variants. Do not expand the entire GE catalogue automatically.
7. Treat a listed GE course as a category candidate, not proof that it is offered in the target term. Keep GE category, Offering status, session, and time-conflict results separate.
8. Do not count a Level 3 project/course toward Free Electives. If the Handbook or programme rules permit double-counting toward a major, minor, or concentration, disclose the consequence and preserve the required GE units.

## Handle follow-up conflict checks

For a follow-up such as “which WPEX courses do not conflict with my required courses”:

1. Reuse the current Offering term and the exact selected/provisional anchor sessions.
2. If the anchor sessions are not fixed, state that the result is provisional or show the result for each viable anchor timetable.
3. Query the supplied concrete codes first. If a supplied code belongs to a Handbook-fixed course, treat `NO_RECORD` as a blocking follow-up and query only the exact variants recorded in the expansion map. For elective or exploratory candidates, expand only when the student explicitly asks or the course has been selected for the plan.
4. Compare every returned time slot against every anchor time slot on the same day.
5. Return a compact result grouped by concrete course and session, including the conflicting course and time when applicable.
6. If the student changes an anchor session, recompute the conflict results; do not reuse the old timetable.

Do not restart the entire planning workflow for a local follow-up. Preserve the student's selected context and query only the changed or newly supplied courses.

## Select and arrange courses

Follow the detailed rules in [references/planning-rules.md](references/planning-rules.md). In particular:

- Anchor the fixed major courses before considering electives.
- Select only the number of elective courses needed to satisfy the Handbook's stated units or slots unless the student asks for alternatives.
- Use Offering data to choose a session, not merely to assert that a course exists.
- Check time-slot overlap using day and minute ranges. A missing or unparsable time is an uncertainty, not proof of no conflict.
- FYP (Final Year Project) courses generally do not have a fixed teaching timetable. If an FYP course has no `timeSlots`, keep it in the plan and label its timetable as project/supervisor-arranged; do not treat the missing time as either a conflict or proof of no conflict.
- Add other courses only after anchor and elective choices are stable.
- Preserve multiple viable plans when session conflicts make the choice subjective.

Do not use `search_planning_courses`; it is not an available capability and the system has no natural-language search service. If the student expresses a topic such as “AI-related”, inspect the returned course names and descriptions as an Agent-level interpretation, and say that it is a best-effort interpretation rather than an exhaustive search result.

## Handle course codes and known variants

`get_course_offerings` accepts exact course codes only. Distinguish these inputs:

- A concrete code such as `WPEX20238001`: query it directly.
- An original course or category code such as `WPEX2023`, `WPEX2033`, or another course code: query the supplied code first. If it is a Handbook-fixed course and returns `NO_RECORD`, look up the original code in `course-code-expansion-map.md` and query only its recorded variants. Do not automatically expand an unselected catalogue candidate.
- A pattern containing the school placeholder convention such as `WPEX2023800X`: look up the base code in the map and replace `X` only with the recorded suffixes. Never send the literal `X` to the MCP tool.

### Data-driven expansion

The course-code map records observed relationships between original codes and concrete Offering codes. It is the authority for whether and how to expand a Handbook-fixed code during this workflow.

- If a Handbook-fixed original code is present in the map, query all of its listed concrete variants in the same calendar term.
- If the original code is absent from the map, do not append `8001` through `8009`, do not guess a suffix, and do not treat the absence of a mapping as proof that the course is unavailable. Report that no known expansion is recorded and ask for an updated CourseDB list or an exact concrete code when further checking is needed.
- If a listed variant has `NO_RECORD`, report that specific variant as missing; do not derive another suffix from it.
- Keep the original-to-variant mapping visible in the result, and keep each variant separate because course names, classifications, or units may differ.

For multiple Handbook-fixed codes with `NO_RECORD`, collect their mapped variants and batch the exact codes in Offering requests of at most 50. When the student provides a new CourseDB list, update the map as observed data instead of adding a generic expansion rule.

`NO_RECORD` for a concrete code means CourseDB has no record for that course and calendar term. For a Handbook-fixed original code, check the map for known variants before reporting the unresolved status.

## Distinguish facts from judgement

Label these as CourseDB facts:

- Handbook requirement type, units, course codes, notes, and course patterns.
- Course name, units, description, prerequisite text, and exclusion text.
- Offering existence, session identifier, time slots, locations, and lecturer names.

Label these as Agent reasoning:

- Why a course is a good fit for the student's stated preference.
- Which elective to prefer when multiple courses satisfy the same requirement.
- Whether a schedule is the most balanced or convenient option.
- Interpretation of prerequisite text when the data is not machine-validated.

The `ME(<programCode>)` catalogue is evidence that a course is classified as a major elective. It is not by itself proof that the course can fill a `Free Elective`, `GE`, `WPEX`, or another Handbook category. Label such a course as a candidate and preserve the eligibility uncertainty unless the Handbook or selection system explicitly establishes the mapping.

Treat `NO_RECORD` for a concrete course from `get_course_offerings` as “CourseDB has no record for this course and term”. For any original course code, consult the expansion map before reporting the unresolved result; only mapped variants may be queried. Do not say that the school definitively cancelled or will not offer the course. Treat raw prerequisite and exclusion text as unverified unless the tool explicitly provides a validated result.

## Return a usable plan

Use this structure:

```text
Planning context
- Program and code:
- Admission/cohort year:
- Current study year:
- Target Handbook study year and term:
- Offering term:

Recommended plan
1. Anchor major courses
   - Course code — course name — requirement source/status — selected session — time
2. Major electives
   - Course code — course name — requirement source/status — why it fits — selected session — time
3. Other courses
   - Course code — course name — requirement source/status — selected session — time

Checks and uncertainties
- Handbook units covered:
- Known time conflicts:
- FYP/project courses without fixed teaching times:
- Offering records not found:
- Original codes with known concrete variants:
- Missing or unverified prerequisite information:

Alternatives
- Include only viable alternative sessions or elective choices.
```

Do not claim that the plan is officially approved, that prerequisites are satisfied, or that a course is offered when the returned evidence is `NO_RECORD`.
