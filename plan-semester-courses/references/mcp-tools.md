# CourseDB MCP tools

Use the connected MCP server at `/api/mcp`. Do not invent REST endpoints or use shell requests during student planning.

## Connection and results

The server uses MCP `2026-07-28` stateless POST requests. The client must support this revision. Legacy `initialize`, sessions, and GET event streams are unsupported.

Each request needs authentication through `Authorization: Bearer <key>` or `x-api-key`. Developer access, key expiry/revocation, API mode, MCP enablement, origin restrictions, and rate limits still apply.

The HTTP headers `MCP-Protocol-Version` and `Mcp-Method` must match the request. Tool calls also need a matching `Mcp-Name`. Put the protocol version and client capabilities under `params._meta`:

```json
{
  "io.modelcontextprotocol/protocolVersion": "2026-07-28",
  "io.modelcontextprotocol/clientCapabilities": {}
}
```

Read `structuredContent` first. If a client only exposes text content, parse its JSON. An `isError` result indicates a failed operation. For query tools, it does not mean `NO_RECORD` or an empty catalogue. HTTP `401`, `403`, `410`, `429`, and `503` indicate access, enablement, or rate-limit failures. Keep retrieved facts and explain the incomplete read.

These tools do not read a student's private history or Planner. A developer key identifies the integration, not the student.

## Shared course fields

Handbook courses, alternatives, and elective catalogue entries use these fields:

- `courseCode`, `courseName: {en, zh}`. The Chinese name can be null.
- `units`, `department`, and `faculty`. The department and faculty can be null.
- `prerequisiteText`, `exclusionText`, and `description`. These can be null.

`get_course_details` returns the same fields plus `classification`. Offering results contain course names at course level, not inside each session. They do not supply units.

## `get_handbook_term_requirements`

```json
{
  "programCode": "CST",
  "cohortYear": 2024,
  "studyYear": 2,
  "studyTermCode": "1"
}
```

Use the exact stored program code and admission cohort. Years range from 2000 to 2099. `studyYear` ranges from 1 to 4.

Handbook term codes are `1` Fall, `2` Winter, `3` Spring, and `4` Summer. They do not determine the student's year level or actual calendar year.

The output includes `handbook` identity, program, cohort, duration, and total units, plus `studyTerm`, `sections`, and `courseLists`.
Each section contains `sectionName` and `requirements`.

`courseLists` contains the first page of school lists bound to that exact Handbook ID.
It uses the `list_course_lists` response fields with `page: 1` and `pageSize: 50`.
Read its list IDs with `list_course_list_courses`.
If `courseLists.hasMore` is true, continue `list_course_lists` with `handbook.handbookId`, the next page, and `pageSize: 50`.
Do not repeat discovery by program and cohort after a successful Handbook read.
If `courseLists.total` is zero, the Handbook has no accessible bound school lists.
Bindings cover the whole Handbook, rather than the requested study term or an individual requirement.

Each requirement supplies `requirementId`, `requirementType`, `requiredUnits`, `isFlexible`, `coursePattern`, `notes`, `course`, and `alternatives`. Preserve these fields. A null course alone does not prove an elective slot. The course can also be unavailable to this read. Use the requirement's type and pattern.

A missing active Handbook produces a tool error. Report that the request failed. Do not diagnose the cause from the generic error text.

## Course list discovery

After reading a Handbook, use its returned `courseLists` as the candidate-list source.
For standalone discovery without a Handbook read, use `list_course_lists` with `programCode` and `cohortYear`.
Read each selected list with `list_course_list_courses`.
`list_course_lists` accepts optional `query`, `handbookId`, `programCode`, `cohortYear`, `courseCode`, `page`, and `pageSize`.
`handbookId` must be a UUID and scopes results to one exact active Handbook.
A missing or inactive Handbook ID returns an empty page.
`query` matches literal list names in both languages.
`courseCode` finds lists containing that exact visible course.
It returns `lists`, `page`, `pageSize`, `total`, and `hasMore`.
Each list supplies its ID, bilingual name and description, ISO `updatedAt`, visible `courseCount`, and active `handbooks`.
Each binding supplies `handbookId`, `programCode`, `programName`, `cohortYear`, and nullable `faculty`.

`list_course_list_courses` requires a UUID `listId` returned by a Handbook read or list discovery.
It accepts optional `query`, `calendarSeason`, `page`, and `pageSize`.
`query` matches literal course codes and names in both languages.
It returns `recordStatus`, nullable `list`, `membershipSource: "COURSE_LIST"`, `seasonFilter`, `courses`, and pagination fields.
Each course supplies `courseCode`, `courseName`, `units`, and nullable `sessionCount`.
It orders courses by code, then ID.
Hidden and versionless courses remain inaccessible.

A missing or private list returns `NO_RECORD`, null metadata, empty courses, zero total, and `hasMore: false`.
An accessible list returns `FOUND` even when the filtered page contains no courses.
Without a season filter, `seasonFilter` and `sessionCount` are null.
With one, `seasonFilter` contains `calendarSeason` and `yearScope: "ALL_YEARS"`.
Each count covers the selected season across all V2 years.

Both tools accept pages from 1 to 10000 and page sizes from 1 to 50.
Defaults are page 1 and page size 20.
Queries contain at most 255 characters.
Program codes contain at most 50 characters, and course codes contain 1 to 16 characters.
Cohort years range from 2000 to 2099.
Follow relevant pages while `hasMore` is true.
If totals change or a request fails, report incomplete coverage.
Preserve list IDs, names, and active Handbook bindings as evidence.

List membership and Offering classification are separate facts.
A binding connects a whole Handbook to a list, rather than individual requirements.
The optional season filter covers all years.
For actual term availability, use `get_course_offerings` with a calendar year and season.

## Free Elective catalogue

Major candidates come from the Handbook's bound course lists.
Use `get_course_details` and target-term Offering tokens for their classification evidence.

Call `list_free_elective_courses` for the exact code inside `FE(...)`:

```json
{ "subjectCode": "ALL", "page": 1, "pageSize": 50 }
```

The tool accepts optional `courseCodePrefix` and `nameContains` literal filters.
`page` ranges from 1 to 1000.
`pageSize` defaults to 50 and cannot exceed 100.

It returns shared course fields, `page`, `pageSize`, `total`, `hasMore`, and `classificationSource: "OFFERING_LATEST_CLASSIFIED_TERM"`.
It also returns `subjectCode`, `classification: "FREE_ELECTIVE"`, and `classificationPattern`.

Follow all relevant pages while `hasMore` is true.
If page limits or failures prevent completion, report incomplete candidate coverage.

`FE(ALL)` matches that exact token.
It does not combine every `FE(...)` catalogue.
For multiple Handbook patterns, query each pattern and deduplicate course codes.

Catalogue entries do not expose the classification term or all type tokens.
Retrieve course details when those facts are needed.

## `get_course_details`

```json
{ "courseCodes": ["COMP2001", "WPEX20238001"] }
```

Send 1 to 50 exact codes, each at most 16 characters. The server trims, uppercases, and deduplicates codes. It returns one `courses` item per normalized code in request order.

Each item contains `courseCode`, `recordStatus`, and `course`. A `FOUND` item contains the shared course fields and `classification`. A `NO_RECORD` item has `course: null`. Missing, hidden, and versionless courses use the same missing shape.

Classification contains `term: {year, semester} | null`, `offeringUnits`, `offeringProgrammes`, `curriculumTypes`, `electiveTypes`, and `typeTokens`. It matches the Developer API course classification.

The term is the latest Offering term with classification metadata. Its classification aggregates every session in that term. A newer unclassified term does not replace it. This is not necessarily the planning term.

Use detail units for selected concrete codes. If they differ from Handbook requirements or cached facts, report the difference. Never borrow a base code's units for its mapped variant.

## `get_course_offerings`

```json
{
  "courseCodes": ["COMP2001", "COMP3004"],
  "calendarYear": 2026,
  "calendarSeason": "FALL"
}
```

Send 1 to 50 exact codes. Years range from 2000 to 2099. Seasons are `SPRING`, `SUMMER`, `FALL`, or `WINTER`. Codes are trimmed, uppercased, and deduplicated.

The output contains `term: {calendarYear, calendarSeason}` and `courses`. Each course contains `courseCode`, optional course-level `courseName`, `recordStatus`, `sessionCount`, `sessionsTruncated`, and `sessions`.

`FOUND` means that the visible course has sessions in the requested term. `NO_RECORD` means that this read found no accessible record. Hidden and versionless courses expose no name. A visible course without target-term sessions can still expose its name.

The tool returns at most 100 sessions per course. `sessionCount` is the full term count. If `sessionsTruncated` is true, continue after the last returned `{session, id}` with the paging tool.

## `list_course_offering_sessions`

```json
{
  "courseCode": "COMP2001",
  "calendarYear": 2026,
  "calendarSeason": "FALL",
  "pageSize": 50,
  "after": { "session": "S100", "id": "4bec6c63-93c7-49bb-a0b5-d19a5c33bf63" }
}
```

Omit `after` for the first page. `pageSize` defaults to 50 and ranges from 1 to 100. A cursor has `session` of at most 50 characters and nonempty `id` of at most 36 characters. Copy it exactly, including whitespace and case.

The output contains `term`, `courseCode`, optional `courseName`, `recordStatus`, `sessionCount`, `sessions`, and `nextCursor`. It uses the same session fields as the batch tool. The database orders sessions by `session`, then `id`.

`nextCursor: null` marks the last page. A page after the final record can have no sessions while its course remains `FOUND`. Record status describes the term, not whether this page is empty.

To complete a truncated batch, keep its sessions and continue from its last session. Use the same course and term on every page. Deduplicate by session ID. Compare each page's total with the initial total.

Stop automatic pagination if a cursor repeats, a page fails, or totals change. Keep the data already read and report incomplete analysis. If unique session count differs from `sessionCount` at the end, report that mismatch too.

Counts and cursors do not establish a snapshot across requests. A concurrent import can change session data without changing its count. Describe results as observed CourseDB data.

## Session fields and interpretation

Each session supplies:

- `id`, `session`, and nullable `schedule`.
- `offeringUnit`, `offeringProgramme`, `curriculumType`, `electiveType`, and `typeTokens`.
- Nullable `requirementsRaw` and `remarksRaw`.
- Nullable parent `rawLecturer`, `lecturers`, and `hasAssignedLecturer`.
- `timeSlots` with `sequence`, nullable `day`, `startMinutes`, `endMinutes`, `rawTime`, `location`, and `locations`.

A lecturer row contains `id`, nullable `lecturerId` and `lecturerName`, `rawLecturer`, and `sequence`. Display rows in sequence order. Prefer each resolved name, then its raw name. If there are no rows, use parent raw text. Do not split names or invent lecturer bindings. `hasAssignedLecturer` reports a binding, not complete teacher information.

Use target session `typeTokens` to compare its classification with the required category. Do not substitute latest-term classification when target tokens are missing or contradictory. Keep fulfillment pending if the evidence does not support the category.

Keep raw requirements and remarks visible when they affect a choice. They do not prove eligibility. Unknown times do not prove freedom from conflicts. A recorded FYP session without slots can remain project-arranged.

Only modern Offering tables supply these results. Empty data never falls back to legacy tables or course-version `type`. `NO_RECORD` does not prove cancellation.

For mapped codes, read [course-code-expansion-map.md](course-code-expansion-map.md). A mapping supports a lookup, not automatic equivalence. Use real session IDs in [Timetable JSON](timetable-json.md).

## `create_timetable_preview`

After selecting the primary recommendation, build the Timetable object under [the JSON rules](timetable-json.md). Call the tool with `{requestId, name, data}`. Generate a UUID for `requestId`. Reuse it with identical arguments for a retry.

The tool returns `structuredContent` with `previewUrl`, `expiresAt`, and `remaining`. The link works for 24 hours after creation. Anyone with the link can preview it. A signed-in viewer can save a personal copy and then edit that copy. The tool does not save a personal timetable or enroll courses.

All keys belonging to one user share 20 successful creations per rolling 24 hours. Reads and retries do not extend expiry. The full MCP request must fit within 64 KiB. Do not truncate data to fit the request.

`INVALID_TIMETABLE` includes field paths. `PREVIEW_REQUEST_CONFLICT` means that the same UUID was used with different content. `PREVIEW_QUOTA_EXCEEDED` includes `retryAfterSeconds`. `PREVIEW_STORAGE_UNAVAILABLE` means that storage failed. Keep the JSON fallback when creation fails. Never work around a quota by changing keys.
