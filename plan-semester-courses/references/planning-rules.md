# Semester planning rules

Apply these rules after loading the Handbook and before recommending sessions.

## 0. Establish historical exclusions first

Before selecting from any ME/FE catalogue, read `references/history-and-exclusions.md` and establish the student's completed, earlier planned/enrolled, reattemptable, and uncertain course sets.

- Exclude completed courses before judging their relevance, topic fit, classification, or Offering. A course can be both an earlier major-required course and a current `FE(ALL)` candidate; its history still makes it unavailable for the new plan.
- Exclude earlier planned/enrolled courses by default when the earlier plan precedes the target term. Keep this separate from proof of completion.
- Do not hard-exclude failed, withdrawn, or dropped courses without the student's confirmation.
- If the student has not provided history and the target contains elective slots, do not turn an elective catalogue entry into a final recommendation or a fully confirmed timetable JSON.

## 1. Anchor major courses first

Treat a Handbook requirement as an anchor when:

- `requirementType` is `REQUIRED` or `CORE`; and
- it has a concrete `course` or a clearly listed fixed alternative.

Put these courses at the front of the candidate plan. A fixed course is not optional merely because its Offering has multiple sessions; the choice is between sessions, not between taking and skipping the course.

If a requirement has `alternatives`, keep the alternatives visible. Do not silently choose one unless the student has given a preference or only one alternative has a usable Offering.

## 2. Fill major electives second

Treat a requirement as an elective slot when it is an `ELECTIVE` requirement without a fixed course, or when its course catalogue is represented by `coursePattern`.

Use the course lists returned by the exact Handbook as the initial candidate pool.
For explicit ME patterns, use `list_major_elective_courses` when classification evidence is needed.
Preserve the scope difference between cohort lists and classification catalogues. The classification source is the latest classified Offering term, such as `ME(CST)`. Use the results as candidates. The classification does not establish permission under a specific cohort's Handbook.

Choose only enough elective courses to satisfy the Handbook's stated `requiredUnits` or number of slots. If multiple courses satisfy the same slot, compare their descriptions, units, prerequisite text, Offering sessions, and the student's stated interests. Keep a small set of alternatives when the choice is subjective.

Do not select the whole major-elective catalogue. Do not infer an elective count from the number of returned courses.

## 2a. Fill Free Electives

Treat a Handbook requirement as a Free Elective requirement when its exact `coursePattern` is `FE(ALL)` or `FE(<subjectCode>)`, such as `FE(SAI)`.

- Start with the lists returned by the Handbook and preserve their membership evidence.
- For an explicit FE pattern, call `list_free_elective_courses` when classification evidence is needed.
  Use the code inside the pattern: `ALL` for `FE(ALL)`, `SAI` for `FE(SAI)`, and so on.
- Match the exact classification. `FE(ALL)` means the `FE(ALL)` catalogue; it is not a request to merge every `FE(...)` classification unless the Handbook explicitly provides multiple patterns.
- Read all pages when `hasMore` is true. `courseCodePrefix` and `nameContains` are literal filters, not semantic search.
- Use the returned courses as the candidate pool, then query their exact course codes with `get_course_offerings` for the target calendar term.
- Select only enough Free Elective courses to satisfy the Handbook's stated units or slots. Do not recommend the entire catalogue.
- A course's `FE(...)` classification is CourseDB evidence, but it does not validate prerequisites, exclusions, or the student's graduation eligibility.
- If a course is classified as both `ME(...)` and `FE(...)`, preserve both facts and avoid double-counting it unless the Handbook or selection system explicitly permits that use.

Do not use `list_major_elective_courses` as a substitute for an `FE(...)` query. A major-elective catalogue is not automatically a Free Elective catalogue.

## 3. Add other courses last

After anchors and major electives are stable, consider remaining Handbook requirements or explicit student requests. Keep these in a separate “other courses” section so the student can see which choices are central to the major and which are supplementary.

If the Handbook data does not identify a course as required, core, or elective, label the category as unresolved instead of inventing a category.

## 3a. Handle GE categories

- Read `references/ge-programme.md` when a Handbook requirement or student request involves GE.
- Preserve the official GE level and category. A generic `GE` label is not enough to decide whether a course belongs to Level 1, Level 2, or Level 3.
- Check the required GE unit pattern: Level 1 is one course in each of three categories; Level 2 is two courses under one or two themes; Level 3 is one course under any one capstone category.
- Use the official GE catalogue for candidate generation, then use Offering data for actual availability and timetable selection.
- Do not treat a GE catalogue entry as offered, prerequisite-satisfied, or eligible for a different Handbook category without supporting data.

## 4. Choose Offering sessions

For every course that may enter the final plan:

1. Query its Offering data for the actual calendar year and season.
2. Discard no course solely because the record is missing; mark it `NO_RECORD` and explain the uncertainty.
3. For `FOUND` courses, complete truncated results with `list_course_offering_sessions` before claiming full comparison.
4. Choose at most one session per course.
5. Prefer a session with complete day, start, end, and location data when other factors are comparable.
6. Preserve alternatives when two sessions are both viable.

Do not assume that session `A` is better than session `B`. Use the student's preferences, complete schedule information, and conflict checks.

When a candidate pool is already known, do not first test a few arbitrary courses and then repeat the request for the full pool. Filter the pool using the Handbook requirement, units, literal preferences, and available course metadata, then query the remaining exact course codes in batches of at most 50. If the student explicitly asks which courses in the full pool are offered, query the full pool in those batches.

### FYP and project-based courses

- Courses identified as FYP (Final Year Project) or an equivalent final-year project generally do not have a fixed teaching timetable.
- A `FOUND` FYP Offering with no `timeSlots` is not missing schedule data by itself. Keep the course as offered, mark it as “project/supervisor-arranged; no fixed class time”, and do not discard it.
- Do not report a missing FYP timetable as a known time conflict or as proof that the course has no conflict. The timetable status is unknown. Ask the student to confirm supervisor or project-meeting arrangements.
- If the course has explicit `timeSlots`, use those slots in the normal conflict calculation.

## 5. Detect known time conflicts

For each pair of selected sessions, compare time slots that have the same day. Two slots overlap when:

```text
first.startMinutes < second.endMinutes
and second.startMinutes < first.endMinutes
```

Report the course codes, sessions, day, and overlap window. Adjacent intervals do not conflict.

Only compare recognized weekdays and integer minute ranges within 0 to 1440, with end later than start. Unknown weekdays and invalid ranges leave conflict status unknown. If explicit slots are absent, retain raw schedule text and its uncertainty. Do not infer times from ambiguous text. If either time is missing or represented only by unparsed raw text, report “conflict status unknown” rather than assuming no overlap.

Do not compare different calendar terms as if they were in the same timetable. Do not treat a shared location as a conflict by itself.

For a follow-up conflict question, preserve the selected/provisional anchor sessions from the current plan and compare only the new exact course codes. If the anchor sessions are not fixed, label the result provisional or evaluate each viable anchor timetable separately.

## 6. Check units and prerequisites carefully

- Read `get_course_details` for selected codes to confirm current units. Sum those units separately from Handbook required units.
- Preserve differences between current course units and Handbook required units. Query concrete variants separately. A mapped base code does not supply variant units.
- Distinguish latest classified-term course metadata from target-session `typeTokens`. If target classification is missing or conflicts, keep elective fulfillment pending.
- Count a course toward a requirement only when its mapping has support. Do not count a course twice without explicit rules.
- Do not turn a system workload suggestion into an institutional rule.
- Present `prerequisiteText` and `exclusionText` as source text.
- Do not claim that the student satisfies prerequisites unless a separate validated service provides that result.
- If a course has missing units or unclear requirement mapping, flag it instead of silently filling the gap.
- A course classified as `ME(<programCode>)` is a major-elective candidate, not automatic proof that it satisfies a Free Elective, GE, WPEX, or other Handbook category.
- A Handbook-fixed course/category code with `NO_RECORD` must first be checked against `course-code-expansion-map.md`. Query only the concrete variants recorded for that original code before finalizing the plan. If no mapping exists, do not force an `800X` expansion or report the course as definitively unavailable. Do not send a placeholder literally. Do not automatically expand an unselected elective catalogue candidate.

## 7. Rank alternatives transparently

When several plans are possible, prefer this order unless the student says otherwise:

1. Covers all fixed anchor requirements.
2. Covers the needed elective units without unnecessary extras.
3. Has no known time conflicts.
4. Uses sessions with complete schedule and location data.
5. Matches the student's explicit preferences.
6. Keeps the workload reasonably distributed.

The last criterion is judgement. Explain it as a recommendation, not as a Handbook rule.

## 8. Treat natural-language preferences as hypotheses

The Agent may interpret statements such as “I want an AI-related elective” by reading the returned course names and descriptions. It must:

- state that this is an Agent interpretation;
- avoid claiming exhaustive topic coverage;
- ask a follow-up question when the candidate set is ambiguous; and
- use exact course codes or literal filters for subsequent MCP calls.
