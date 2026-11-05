---
name: plan-semester-courses
description: Plan a student's semester from CourseDB facts and course history. Compare requirements, elective choices and timetable conflicts. Create a temporary preview only when requested. Use Timetable import JSON when preview creation fails.
metadata:
  version: "2.11"
---

# Plan Semester Courses

Use CourseDB facts to draft a provisional semester plan. Do not write to a Planner or claim official academic approval.
Create a preview only when the current user explicitly requests one or an export.
For "先看" or candidate inspection, return the requested facts and stop before export references or preview calls.

Load resources for the current stage only.
If major or admission year is missing, ask the basics before reading references.
Once the basics are known, read the exact Handbook and target-term Offerings.
Read [planning-rules.md](references/planning-rules.md) before selecting courses.
Read [timetable-json.md](references/timetable-json.md) only before creating a preview or import JSON.

In built-in chat, tool declarations supply argument schemas. Do not read `mcp-tools.md` for the external MCP transport.
Other clients read [mcp-tools.md](references/mcp-tools.md) before calling tools.
Built-in results use `{status: "OK", data}` or `{status: "ERROR", code, instruction?}`. Read facts from successful `data` only.
Course facts use `courseCode`, `courseName: {en, zh}`, `units`, and nullable description, prerequisite and exclusion text.
Offering names are at course level, not inside sessions. Offerings supply schedules, not units.
Tools do not read private student history. Do not treat a timetable or Handbook as completion evidence.

In built-in chat, read references with `read_skill_reference` and skill `plan-semester-courses`. Other clients read the linked files.
Read each required reference once while its content remains in the conversation.
Wait for each required read to succeed before calling a dependent tool. Do not submit a reference read and its dependent query together.
If a reference read fails, report that failure and stop dependent queries.
If a tool rejects its arguments, correct them and retry without mentioning the failed call to the student.
If a tool fails, report its actual effect without inventing a cause.
Do not interpret a failed read as a missing Handbook, empty requirements, or unavailable courses.

Load these references when their conditions apply:

- If `find_handbooks` leaves the program code unresolved, read [program-code-map.md](references/program-code-map.md). Reuse a code from the picker or a successful lookup.
- If the plan includes GE, read [ge-programme.md](references/ge-programme.md). Preserve its levels, categories, and unit rules.
- If an Offering returns `NO_RECORD` or a code contains `800X`, read [course-code-expansion-map.md](references/course-code-expansion-map.md).
- Before selecting electives, read [history-and-exclusions.md](references/history-and-exclusions.md).
- For worked examples and failure cases, read [planning-examples.md](references/planning-examples.md).

## Talk with the student

Students describe themselves by major, admission year or grade (大几), and the term they want to plan.
Use those terms in every reply.
Never show or ask for internal fields such as `studyYear`, `studyTermCode`, `cohortYear`, `calendarSeason`, term codes, Handbook years or terms, tool names, requirement IDs, or error codes.
Never ask the student which Handbook to use.

Ask in short rounds, not as one long list.
In built-in chat, ask with `ask_student`; other clients ask in plain text with short numbered options.
Skip any question the student has already answered, and never ask for the same fact twice.

1. Basics: target term, major, and admission year.
   Offer concrete terms from the calendar context, such as "2027 春季（下学期）".
   In built-in chat, ask for the major and admission year with one `ask_student` question of kind `handbook`.
   It opens CourseDB's program picker; the answer names the program, its program code, and the admission year. Use that code and year directly.
   Never offer a short list of majors yourself. Other clients let the student type the major and map it with the program code map.
2. History and preferences, after the basics are settled: whether earlier terms followed the standard plan, courses taken outside it or failed, and preferences such as interests, free days, early classes, or course load.
   Offer an option such as "按培养方案正常修读" and accept free text for exceptions.
3. Elective choices, after anchors and candidates are read: see "Offer elective choices".
4. Ask again only when a real choice remains, such as a time conflict.

Start reading CourseDB as soon as the basics are known. You may read the Handbook while asking about history.
If the student's first message already contains enough facts, plan without asking.

## Establish context

Derive the Handbook lookup yourself.
In built-in chat, the server supplies a calendar context with today's date, the meaning of "this semester" and "next semester", and the Handbook coordinates for each admission year and term.
Use that table for `cohortYear`, `studyYear`, and `studyTermCode`, and use the term itself for `calendarYear` and `calendarSeason`.
Without that context, the academic year starts in Fall: `studyYear` is the academic start year minus the admission year plus one.
Handbook term codes are `1` Fall, `2` Winter, `3` Spring, and `4` Summer.
If the result falls outside study years 1–4, or the student follows a different year's plan, ask in plain words, for example "你目前按哪个年级的培养方案选课？".

Use the exact program code from the picker or a successful Handbook lookup. Do not infer it from a course prefix.

Treat the student's history answer as settled for this conversation:

- "Standard plan" means earlier-term required courses are complete. Do not read earlier Handbook terms only to rebuild that history.
- Exclude courses the student says they completed or are taking. Offer retakes only for courses they say they failed or dropped.
- If the student skips history, plan with the available facts and state that assumption once in the final summary.

Do not ask for history again, and do not label each course as uncertain because history is incomplete.
Record history as completed, enrolled, planned, failed, withdrawn/dropped, or unknown. Earlier Timetable entries prove a plan, not completion.

## Read requirements and candidates

1. Call `get_handbook_term_requirements` for the exact program, cohort, and Handbook term. If the tool fails, report the failure. Do not interpret an error as an empty Handbook or substitute another cohort.
2. Preserve each requirement's section, ID, type, `requiredUnits`, flexibility, pattern, notes, fixed course, and alternatives. Establish fixed `REQUIRED`/`CORE` anchors first.
3. Use `courseLists.lists` returned by the Handbook query. Read the selected IDs with `list_course_list_courses`. If `courseLists.hasMore` is true, continue `list_course_lists` with `handbook.handbookId` and the returned page size. Follow the member pages while `hasMore` is true. Preserve list membership separately from requirement categories. Apply history exclusions before selecting candidates.
4. For explicit ME patterns, read candidate classification through `get_course_details`. Compare target-session tokens separately. For an explicit `FE(...)` catalogue request, use `list_free_elective_courses`. Explain its broader scope. For category patterns and GE categories, use `find_term_courses` as described in "Find category candidates".
5. Build an exact candidate list. Use GE and observed code maps only when relevant. Query at most 50 codes per batch.
6. Call `get_course_details` for missing course facts and for selected codes whose current units or classification need confirmation. Use each concrete variant's own facts.
7. Call `get_course_offerings` for the explicit calendar term. If results are truncated, continue with `list_course_offering_sessions` as described in the tool reference.
8. For a Handbook-fixed code with `NO_RECORD`, query every recorded concrete variant before finalizing. Query variant details as well. If no mapping exists, report the unresolved code.

Keep a checklist of every fixed `REQUIRED` or `CORE` course, including university core courses.
Read target-term Offerings for each anchor before choosing supplementary courses.
`get_course_details` supplies no sessions and cannot establish target-term availability.
Do not interpret its missing session field as an empty Offering result.
If an Offering read fails, keep that anchor unresolved and follow the error's recovery instructions.
For `TOOL_RESULT_TOO_LARGE`, use the returned `recovery.toolName` and `recovery.arguments` when present.
For an oversized Offering batch, page each requested code with `list_course_offering_sessions` in the original term.
Start with `pageSize: 5` and no cursor because a failed batch supplies no session data.
Follow successful page cursors and preserve earlier results.
If one page is still too large, reduce its page size without changing its cursor.
For `TOOL_BUDGET_EXHAUSTED`, stop tool reads and report incomplete coverage.
Do not treat either size error as `NO_RECORD` or defer a required course to a later term without evidence.

Preserve Handbook categories exactly.
A code such as `UCHL1XX3` does not establish a Free Elective category.
Do not replace an unresolved Healthy Lifestyle requirement with an unrelated elective.

If the Handbook returns `courseLists.total: 0`, report that no school list is bound to that Handbook. Keep Handbook anchors and explicit alternatives available.
For broader major candidates, use explicitly requested school lists and state their scope.
For broader Free Elective candidates, use the exact FE catalogue requested by the student.
Do not silently substitute another cohort.

Do not use historical season counts as target-term availability.
List membership alone does not establish elective fulfillment.

When combining bound lists, deduplicate candidates by exact course code.
Preserve every source list for each candidate.
Use `query` for literal course-code or bilingual name searches within a list.
Interpret topic preferences through the returned names and descriptions.
Do not claim complete topic coverage from a literal keyword search.
If pages fail or totals change, keep the retrieved candidates and report incomplete coverage.
For availability across the full candidate pool, query every relevant code in the actual calendar term.
Do not narrow that request with the historical season filter.

Do not invent suffixes, wildcard queries, or a natural-language search tool. For exploratory courses, expand only selected candidates or codes that the student explicitly asks to expand.

## Find category candidates

Some requirements name a category instead of a course: a code pattern such as `UCHL1XX3`, a GE category such as Quantitative Reasoning, or a major or free elective pool.
Enumerate category candidates from the target term with `find_term_courses`:

- For a code pattern, pass it as `courseCodePattern`, for example `UCHL1XX3`. It matches whole codes, so restricted variants such as `UCHL12038001` are excluded.
- For a classification, pass the exact session token as `typeToken`, for example `UCHL(ALL)`, `GFQR(ALL)`, `ME(CST)`, or `FE(ALL)`.
- Follow `hasMore` pages until you have enough candidates that fit the timetable, or the pool is exhausted.

Do not use `course-code-expansion-map.md` to enumerate a category. Its variants are separate courses, and some are restricted sections, for example for students with special health needs.
Read a candidate's description before offering it, and skip courses whose description restricts enrolment.
Never report that only a few courses exist before `find_term_courses` has been read for the target term.

## Offer elective choices

Fixed `REQUIRED` and `CORE` anchors need no choice. For every other slot with real options, let the student choose:

1. Shortlist 2–4 candidates that are offered in the target term, fit the anchors without conflicts, and match the student's preferences.
2. Ask with `ask_student`: each option label is the course code and short name; its description gives one concrete reason, such as the free day it keeps, its topic, its time, or its workload.
3. Add an option that lets you decide, such as "你帮我选". Ask about up to three slots in one call.
4. If only one candidate fits, explain why and use it without asking. If the student already asked you to decide, choose and name the alternatives briefly.

After the student chooses, build a preview only when requested. Do not present a complete plan with electives already chosen before asking.

## Select sessions and explain evidence

Arrange anchors first, major electives second, Free Electives third, and remaining courses last. Select at most one session per course. Preserve usable alternatives when the choice depends on preferences.
Keep alternative sessions separate from selected sessions. If the choice is Monday or Tuesday, never describe both days as occupied by that course.

Latest classified-term course metadata supplies candidate classification. Target-term session metadata supplies that session's classification. If they conflict or the target classification is missing, keep elective fulfillment pending.

Use current course-detail units for selected courses. Preserve differences from Handbook required units. A code mapping does not establish equal units or institutional equivalence.

Read names from the course-level `courseName`. Read linked lecturers in sequence order, with resolved names before raw names. Keep `requirementsRaw` and `remarksRaw` as source text.

Compare complete same-day time intervals. Adjacent intervals do not overlap. Missing or invalid times leave conflict status unknown. A recorded project session without fixed times can remain provisional.

Do not infer prerequisites, graduation eligibility, or cancellation from raw text, classification, or `NO_RECORD`. Separate retrieved facts from preference-based ranking and other reasoning.

Recommend course codes and names only from successful CourseDB results or an explicit student-supplied course list.
Read target-term Offering results before assigning any session, time, lecturer, or location.
Never invent courses, session IDs, times, rooms, units, or claims about historical offerings.
Do not provide a typical or illustrative timetable as a fallback for a failed query.
If evidence is missing, report the unknown facts and the failed or pending read.
Keep student-supplied course choices unverified until tools return supporting facts.

## Follow-ups and final output

Keep the current context, requirements, history, details, session IDs, pagination completeness, and unresolved issues across follow-ups. Query only changed or new exact courses. If an anchor changes, recalculate affected conflicts.

Describe the planning context in student terms, recommended courses and sessions, requirement coverage, history exclusions, known conflicts, unknowns, and viable alternatives.
Keep caveats short and grouped at the end. Do not repeat a caveat that the student has already resolved.
Explain failed reads by their effect, such as "some sessions could not be checked", not by tool or error names. Report required units separately from confirmed covered units. Do not double-count a course without explicit supporting rules.

If history is missing, select electives from the available facts and mention once that earlier courses were not checked.
Include all selected sessions in the provisional JSON and preview, including electives with pending requirement coverage.
A preview shows a draft schedule, not academic eligibility or confirmed requirement fulfillment.
If the student asks to ignore prerequisites, honor that choice for planning and preview creation.
Preserve prerequisite source text and state that satisfaction remains unverified.
Do not require history or academic approval before creating the requested preview.

Build strict Timetable JSON when the student requests a preview or export, after any outstanding elective choices. Use real session IDs and the actual calendar term. Keep uncertainties and alternatives outside the JSON. Follow the import limits in the export reference.
If the student asks to inspect candidates or says not to create a preview, stop after answering that question.

Include every selected session with valid import fields, even when its schedule is null or its times are unknown.
Use an empty schedule only for a real returned session whose schedule is null.
Without a returned session ID, list a course as unresolved, not selected or included in the preview.
Before creating a preview, compare the selected codes against every fixed anchor and report each omission explicitly.
The built-in preview result includes `planningSummary.includedCourseCodes` and `planningSummary.unresolvedRequiredCourses`.
Use that summary to describe what the preview actually contains.
Its scope covers exact Handbook anchors only, not all elective requirements or academic eligibility.
Do not describe an omitted course as selected, covered, or included.
Do not claim that an incomplete subset is the complete semester plan.
If preview creation succeeds, return its link without repeating import JSON unless the student asks for JSON.

When the student requests the full preview, use the current full selection rather than reusing an earlier anchor-only link.
If `create_timetable_preview` is available, generate a UUID for `requestId`. Call the tool with this UUID, a short name, and the Timetable object as `data`. Reuse the UUID and identical arguments for a retry. Return the preview link and expiry on success. Explain that anyone with the link can view the draft. A signed-in user can save a personal copy. Do not claim that the tool saves to an account.

If the tool is unavailable or fails, explain the failure and return the strict import JSON. Do not retry a quota error with another UUID or key.
