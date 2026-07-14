# CourseDB MCP tool reference

Use these tools through the connected MCP server. Do not use shell commands or invent REST endpoints when the MCP tools are available.

## Connection and access

- MCP endpoint: `/api/mcp`.
- Authenticate every request with a valid DEV API Key using `Authorization: Bearer <key>` or `x-api-key`.
- The key must be active, unexpired, and owned by a user with developer access.
- If the system MCP gate is off, the endpoint returns HTTP `410 Gone` with code `MCP_DISABLED`.
- If the broader Developer API mode is disabled, the existing Developer API gate may return HTTP `503`.
- Tool failures are MCP tool errors. Treat them as facts about the request failure, not as course-planning results.

## `get_handbook_term_requirements`

Use this first. It accepts:

```json
{
  "programCode": "CST",
  "cohortYear": 2024,
  "studyYear": 2,
  "studyTermCode": "1"
}
```

Inputs are exact and structured:

- `programCode`: the academic program code, such as `CST`.
- `cohortYear`: the student's admission year/Handbook cohort.
- `studyYear`: `1` through `4`.
- `studyTermCode`: one of `"1"`, `"2"`, `"3"`, `"4"`.

The result contains:

- `handbook.handbookId`, `programCode`, `programName`, `cohortYear`, `durationYears`, and `totalUnits`.
- `studyTerm.studyYear` and `studyTerm.studyTermCode`.
- `sections[].sectionName`.
- `sections[].requirements[]` with `requirementId`, `requirementType`, `requiredUnits`, `isFlexible`, `coursePattern`, `notes`, `course`, and `alternatives`.

`course` and `alternatives` contain only planning fields: course code, bilingual name, units, department, faculty, prerequisite text, exclusion text, and description. A null `course` commonly means that the requirement is an elective slot rather than a fixed course.

The Handbook study-term code is not an Offering season. Use the separate calendar term for `get_course_offerings`:

| `studyTermCode` | CourseDB label |
| --- | --- |
| `1` | Fall (Sem 1) |
| `2` | Winter |
| `3` | Spring (Sem 2) |
| `4` | Summer |

## `list_major_elective_courses`

Use this after resolving the program code and Handbook term. It reads courses whose current course-version type contains an exact `ME(<majorCode>)` entry, such as `ME(CST)`.

Example:

```json
{
  "majorCode": "CST",
  "page": 1,
  "pageSize": 50
}
```

Optional filters:

- `courseCodePrefix`: literal course-code prefix, for example `COMP`.
- `nameContains`: literal English course-name substring. This is not semantic search.
- `page`: starts at `1`.
- `pageSize`: maximum `100`, default `50`.

The result includes `courses[]` with course code, bilingual name, units, department, faculty, prerequisite text, exclusion text, and description. It also includes `total`, `hasMore`, `page`, and `pageSize`.

Do not treat the entire returned catalogue as the student's selected courses. Use it as the candidate pool for an elective requirement, then inspect Offering sessions.

## `get_course_offerings`

Use this with explicit course codes and the actual calendar term:

```json
{
  "courseCodes": ["COMP2001", "COMP3004"],
  "calendarYear": 2026,
  "calendarSeason": "FALL"
}
```

Allowed `calendarSeason` values are `SPRING`, `SUMMER`, `FALL`, and `WINTER`. Send no more than 50 exact course codes per call. The tool does not expand prefixes, wildcards, or placeholder characters. When expansion is needed, read `references/course-code-expansion-map.md` and send only the exact variants recorded there.

For each requested course, the result contains either:

- `recordStatus: "FOUND"` and one or more `sessions`, or
- `recordStatus: "NO_RECORD"` and an empty `sessions` array.

Each session may include:

- `session` identifier, such as `A` or `B`;
- display `schedule` text;
- lecturer names;
- `timeSlots[]` with `day`, `startMinutes`, `endMinutes`, raw time, and location fields.

For FYP (Final Year Project) or equivalent project-based courses, a `FOUND` session may legitimately have no `timeSlots` because the work is arranged through project or supervisor meetings. Preserve the Offering result and label the timetable as project-arranged rather than treating it as an ordinary missing timetable.

`NO_RECORD` means CourseDB has no record for that course and term. It does not prove that the institution will not offer the course. Missing time data also does not prove that a session has no conflict.

Examples of input handling:

- Query `WPEX20238001` as a concrete course code.
- Query the original code first, for example `WPEX2023`, `WPEX2033`, or `COMP1003`. If it returns `NO_RECORD`, use only the variants listed for that original code in the expansion map.
- For `WPEX2023800X`, replace `X` only with the suffixes recorded for `WPEX2023`. Never send the literal `X` and never assume every digit from `1` through `9` is valid.

The expansion map is observed CourseDB data, not natural-language search and not a universal numbering rule. Expansion is mandatory for a Handbook-fixed course with `NO_RECORD`; for an unselected elective or exploratory candidate, wait until it is selected or the student explicitly asks. Map each found variant back to its original code in the response.

## Tool boundary

There is no natural-language course search tool. Do not call or mention `search_planning_courses`. Use the Agent's language understanding to interpret the student's request, then use this reference data and deterministic filters. Never present that interpretation as a complete semantic search result.
