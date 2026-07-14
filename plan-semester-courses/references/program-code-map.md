# CourseDB program and course-type abbreviations

Use this map to normalize a student's program name to the exact code expected by the MCP tools. The authoritative source in the application is `src/lib/constants.ts`; update this reference when that source changes. If the student's wording is ambiguous, ask instead of guessing.

## Faculty and program codes

### FST — Faculty of Science and Technology

| Code | Program |
| --- | --- |
| `AM` | Applied Mathematics |
| `FM` | Financial Mathematics |
| `STAT` | Statistics |
| `DS` | Data Science |
| `CST` | Computer Science and Technology |
| `AI` | Artificial Intelligence |
| `FS` | Food Science |
| `ENV` | Environmental Science |
| `APSY` | Applied Psychology |

### SCC — School of Culture and Creative

| Code | Program |
| --- | --- |
| `CCM` | Culture Creativity and Management |
| `MAD` | Media Arts and Design |
| `THEM` | Tourism Hospitality and Event Management |
| `AIM` | Animation and Interactive Media |
| `CTV` | Cinema and Television |
| `GD` | Game Design |
| `MUS` | Music Performance |

### FHSS — Faculty of Humanities and Social Sciences

| Code | Program |
| --- | --- |
| `CCGC` | Chinese Culture and Global Communication |
| `MCOM` | Media and Communication Studies |
| `PRA` | Public Relations and Advertising |
| `ATS` | Applied Translation Studies |
| `ELLS` | English Language and Literature Studies |
| `DGS` | Digital Social Sciences |
| `GAD` | Globalisation and Development |
| `SWSA` | Social Work and Social Administration |

### SGE — School of General Education

| Code | Program |
| --- | --- |
| `CLCC` | Chinese Language and Culture Centre |
| `CFLC` | Centre of Foreign Languages and Cultures |
| `ELC` | English Language Centre |
| `WPEC` | Whole Person Education |

### FBM — Faculty of Business Management

| Code | Program |
| --- | --- |
| `ACCT` | Accounting |
| `FIN` | Finance |
| `AE` | Applied Economics |
| `BUSA` | Business Analytics |
| `MHR` | Management and Human Resources |
| `MKT` | Marketing Management |
| `EBIS` | E-Business Management and Information Systems |
| `EPIN` | Entrepreneurship and Innovation |
| `DMM` | Digital Media Management |

## Course type codes

Course types are stored as comma-separated entries with a major in parentheses, for example `ME(CST), MR(AI)`. Match the whole type entry, not a loose substring.

| Code | Meaning |
| --- | --- |
| `MR` | Major Required |
| `ME` | Major Elective |
| `FE` | Free Elective |
| `GE` | General Education |
| `GCAP` | General Capstone |
| `UC` | University Core |
| `VS` | Volunteer Service |
| `EA` | Environmental Awareness |
| `XA` | Experiential Arts |
| `WPE` | Whole Person Education |
| `UNK` | Unknown |

Examples:

- `ME(CST)` means the current course version is classified as a Computer Science and Technology major elective.
- `MR(AI)` means the current course version is classified as an Artificial Intelligence major-required course.
- `ME(CST), MR(AI)` means both classifications are present.

The program code and the course classification code normally align in the current data model. Still, use the Handbook result as the source for the program code and do not infer eligibility from a course prefix alone.
