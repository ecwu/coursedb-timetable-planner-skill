# Course code expansion map

This reference is derived from the CourseDB course list supplied by the user. It contains 69 observed concrete course records grouped under 55 original course codes.

Use this file as an observed mapping, not as a universal numbering rule. The Agent must use it automatically when a Handbook-fixed course has no direct Offering, and must use only the concrete variants listed here. For an unselected elective or exploratory candidate, consult it only when that course is selected for the plan or the student explicitly asks for expansion.

## Agent procedure

1. Query the original Handbook-fixed course code first.
2. If the original code has a `FOUND` Offering result, do not expand it.
3. If the original code has `NO_RECORD`, look up the original code in the table below.
4. If the table contains variants, query those exact concrete codes in the same calendar term before finalizing the plan. Keep each variant separate because names, course types, units, or eligibility may differ.
5. If the table has no entry, do not generate `8001` through `8009`, do not guess a suffix, and do not send a literal `X`. Report that no known expansion is available and ask for a source list or exact code if the student wants further checking.

When the student provides a pattern such as `WPEX2023800X`, replace `X` only with the suffixes listed for `WPEX2023`. Do not expand a pattern beyond the values recorded in this file.

## Observed mappings

| Original course code | Known suffixes | Known concrete course codes |
| --- | --- | --- |
| `ACCT2023` | `8001` | `ACCT20238001` |
| `AIM2033` | `8001` | `AIM20338001` |
| `AIM3013` | `8001` | `AIM30138001` |
| `AIM3053` | `8001` | `AIM30538001` |
| `CHI1073` | `8001`, `8002` | `CHI10738001`, `CHI10738002` |
| `CHI1103` | `8002` | `CHI11038002` |
| `CHI1203` | `8002` | `CHI12038002` |
| `CHI1273` | `8001`, `8002` | `CHI12738001`, `CHI12738002` |
| `CTV2023` | `8001` | `CTV20238001` |
| `CTV2063` | `8001` | `CTV20638001` |
| `CTV3153` | `8001` | `CTV31538001` |
| `CTV4013` | `8001` | `CTV40138001` |
| `CTV4103` | `8001` | `CTV41038001` |
| `CTV4113` | `8001` | `CTV41138001` |
| `CTV4173` | `8001` | `CTV41738001` |
| `DSS3003` | `8001` | `DSS30038001` |
| `EBIS2013` | `8002` | `EBIS20138002` |
| `ENG1013` | `8002` | `ENG10138002` |
| `ENG2163` | `8001`, `8002` | `ENG21638001`, `ENG21638002` |
| `ENG2183` | `8002` | `ENG21838002` |
| `ENG3003` | `8001`, `8002` | `ENG30038001`, `ENG30038002` |
| `ENG3053` | `8001` | `ENG30538001` |
| `ENG3083` | `8002` | `ENG30838002` |
| `ENG3093` | `8002` | `ENG30938002` |
| `ENG3203` | `8001`, `8002` | `ENG32038001`, `ENG32038002` |
| `ENG3223` | `8001`, `8002` | `ENG32238001`, `ENG32238002` |
| `ENG3313` | `8001`, `8002` | `ENG33138001`, `ENG33138002` |
| `ENG4123` | `8002` | `ENG41238002` |
| `ENG4153` | `8002` | `ENG41538002` |
| `ENG4183` | `8001`, `8002` | `ENG41838001`, `ENG41838002` |
| `GCAP3173` | `8001` | `GCAP31738001` |
| `GFHC1033` | `8002` | `GFHC10338002` |
| `GFHC1073` | `8001`, `8002` | `GFHC10738001`, `GFHC10738002` |
| `GLD2013` | `8002` | `GLD20138002` |
| `HIST1003` | `8002` | `HIST10038002` |
| `MCOM2083` | `8002` | `MCOM20838002` |
| `MKT3033` | `8002` | `MKT30338002` |
| `MKT3063` | `8002` | `MKT30638002` |
| `PRA3063` | `8002`, `8003` | `PRA30638002`, `PRA30638003` |
| `PRA4083` | `8002` | `PRA40838002` |
| `PSY4033` | `8003` | `PSY40338003` |
| `REL1033` | `8002` | `REL10338002` |
| `TESL2003` | `8001` | `TESL20038001` |
| `TESL3033` | `8002` | `TESL30338002` |
| `TESL3063` | `8001` | `TESL30638001` |
| `TESL3083` | `8002` | `TESL30838002` |
| `TRA1003` | `8002` | `TRA10038002` |
| `TRA1013` | `8002` | `TRA10138002` |
| `TRA3043` | `8001` | `TRA30438001` |
| `TRA3133` | `8002` | `TRA31338002` |
| `UCHL1203` | `8001`, `8002`, `8003` | `UCHL12038001`, `UCHL12038002`, `UCHL12038003` |
| `UCHL1213` | `8001`, `8002`, `8003` | `UCHL12138001`, `UCHL12138002`, `UCHL12138003` |
| `WPEX2013` | `8001` | `WPEX20138001` |
| `WPEX2023` | `8001` | `WPEX20238001` |
| `WPEX2033` | `8001` | `WPEX20338001` |

## Interpretation limits

- The suffix is a concrete course-code variant, not a session identifier. Use `get_course_offerings` to retrieve its sessions.
- Different variants of the same original code may have different names or classifications. Never select the lowest suffix automatically.
- This table records what was observed in the supplied list; it does not prove that every listed variant is offered in every calendar term.
- If a new CourseDB export shows a new variant, update this table rather than teaching the Agent to guess a broader suffix range.
