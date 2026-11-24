# School assets

Locate records with school_assets_list by kind, read current values and revision with school_asset_get, and edit existing records with school_asset_save(action=update) (school_asset_update is the older edit-only entry with the same behavior). school_asset_save(action=create) creates scholarships, academic-calendar terms and application rules; it takes no id or expectedRevision. kind values are campuses=campuses（校区）, important-times=academic-calendar terms（学校重要日期）, scholarships=scholarships（奖学金）, videos=videos（视频）, and application-rules=application rules（申请须知）. Use an id from the same school's list of that kind; video IDs are opaque strings.

## Campuses: campuses

| Field | Meaning and related behavior |
| --- | --- |
| campusName | Campus name（校区名称）. If changed without displayCampusName, the API also updates the display name. |
| displayCampusName | Separate display name（显示名称）. To rename a campus while retaining its old display name, explicitly submit that old display name too. |
| state | Australian state/territory abbreviation. Use the eight values in the current schema. |
| city / postcode | City（城市）/ four-character postcode（邮编）. Preserve leading zeroes, for example 0820. |
| addressLine1–4 | Four independent address lines（地址行）. Omission preserves each line; null clears that line. |

All these fields can be cleared with null when editing. Missing source information is not an instruction to clear them. The API rebuilds detailAddress from the address lines, city, state, and postcode and marks the record as manually modified. rawState/rawCampusName/rawDetailAddress/locationKey/isModify/detailAddress are read-only; do not edit source values. The webpage's restore-original-data action is not exposed through this MCP.

Each campus address represents one location; addressLine1–4 are parts of that location, not four separate teaching sites. This MCP edits existing campuses but cannot create locations or assign theory/workshop roles to a course. Verify a specific existing record before correcting it; do not repurpose a different campus, concatenate separate sites, or use names/descriptions to simulate missing relations. For unsupported location work, follow [destination selection](editing.md#choose-a-destination-before-writing) and keep the remaining feedback work unfinished.

## Academic calendars: important-times

| Field | Meaning and constraints |
| --- | --- |
| year / intake | Calendar year / intake name（年份/学期名称）. An intake name must be unique within the same school and year. Calendar years are independent of the course-fee year allowlist. |
| teachingStart / teachingEnd | Teaching start/end（教学开始/教学结束）. Required on the webpage; MCP does not allow clearing them. End cannot precede start. |
| censusDate | The school's published Census Date. Follow the school's definition; do not treat it as an application deadline. |
| examsStart / examsEnd | Exam start/end（考试开始/考试结束）. End cannot precede start. |
| resultRelease | Results release date（成绩公布）. |
| applicationClose | The school's published application deadline for this term（申请截止）. The deadline day itself is still open; the term counts as closed from the next day. Cannot be after teachingStart. Record a deadline only here, not in the intake name, a consultation note or an application rule. |
| appliesToStudents | Who the deadline applies to（截止适用学生）: all=all students（全部学生, default）, international=international students（国际生）, domestic=domestic students（本地学生）. Cannot be cleared. |
| appliesToCourses | Which courses it applies to（截止适用课程）: all=all courses（全部课程, default）, coursework=coursework（授课型）, research=research master's and doctoral degrees（研究型）. Cannot be cleared. |
| mayCloseEarly | true only when the school states that popular programs may close before the deadline once full（可能提前截止）. Default false. |
| sourceUrl | The page where the school publishes these dates（截止日来源链接）. Absolute http(s) URL, at most 1000 characters; null clears it. |
| verifiedAt | The date you checked the dates against sourceUrl（核对日期）, YYYY-MM-DD, no later than today's date in Sydney. Set it only after opening the source page; null clears it. |

Use YYYY-MM-DD dates. year accepts 1900–2200. Teaching, deadline, census, exam and results dates must fall within year−1 through year+1. verifiedAt follows the Sydney date rule above regardless of the term year. Dates other than teaching start/end may be cleared with null; the intake name cannot be cleared. Calendar changes can affect associated course dates and revision, not just this row's display.

### Create terms and record deadlines

A school may have no calendar rows at all. To enter a year from the school's official dates:

1. `school_assets_list(kind=important-times, year=<year>)` and page through every result. If a term with the same intake name already exists for that year, update it instead of creating a duplicate; the API rejects duplicate names in the same school and year.
2. Create each missing term with `school_asset_save(action=create, kind=important-times)`. year, intake, teachingStart and teachingEnd are required; add censusDate, exam dates, resultRelease and applicationClose when the source states them. Use the school's own term name (for example `Term 1`, `Semester 2`).
3. Set the audience the published deadline covers. A deadline the school publishes only for international coursework applicants is `appliesToStudents=international`, `appliesToCourses=coursework`. Keep `all` when the source does not restrict it. When one term has different deadlines for different audiences, create one term per audience and tell them apart by intake name, for example `Term 1` and `Term 1 (Research)`; do not merge two deadlines into one row.
4. Set sourceUrl to the page that states the dates and verifiedAt to the day you checked it. Use a fresh idempotencyKey per term; reuse it with the same payload only when retrying that same create.
5. Read the year back with school_assets_list and compare each term's dates with the source.

The website and public MCP match a course start date to a term with the same teachingStart. A course uses the term's deadline only when the term is for new students (not returningStudentsOnly), has applicationClose, is not domestic-only, and appliesToCourses fits the course (research for research master's and doctoral degrees, coursework for the rest, all for both). A missing or wrong audience therefore hides or misapplies the deadline. A course-specific deadline that differs from the school's term deadline is not a calendar fact; for research degrees use the course's `annual.researchDegree.deadlines`.

## Scholarships: scholarships

| Field | Meaning and related behavior |
| --- | --- |
| name / description | Name / eligibility or usage conditions（奖学金名称/使用条件）. Both are **Chinese source text; write in Chinese**, with other languages derived by the backend. Both are required plain text. description is not course information or a calculation formula. |
| level | Qualification scope（学历范围）: all=all（全部）, diploma=diploma（专科）, bachelor=bachelor's（本科）, master=master's（研究生）. Independent of range. |
| range | Applicability（适用范围）: all=whole school（全部）, subject=subject category（专业）, course=specific course（课程）. |
| subject | Required for range=subject. Two-character classification code; preserve leading zeroes. The webpage calls 00 “全校”, but the public site matches subject code 00 exactly, not as a whole-school wildcard. Use range=all for the whole school. Use the schema for other code meanings; do not guess. |
| courseCode | Required for range=course and must belong to this school. |
| studentLocation | Student location（学生所在地）: onshore=in Australia（境内）, offshore=outside Australia（境外）, all=anywhere（全部）. This is not citizenship. |
| endDate | Deadline（截止日期）. null clears it; a missing or expired date produces websiteVisible=false. |

Changing range clears inapplicable subject/courseCode fields and rebuilds associations. Submitting subject/courseCode alone does not switch range. Do not copy inapplicable fields from a read result back into patch. Scholarship records do not automatically change course tuition or tuitionByScholarship.

websiteVisible and visibilityReason are read-only derived information: missing_end_date means no date; expired means the deadline has passed. Comparison uses the API's current date, and the deadline day still satisfies the date condition. This checks only dates; the public website also filters by course qualification, applicability, and visitor location. It does not prove website synchronization or imply an available publish operation.

## Application rules: application-rules

Application rules（申请须知） are short notes a school publishes about how applications are handled: assessment method, document deadlines, quotas, offer conditions. Each rule is one topic and its points, shown on the school page in sort order.

| Field | Meaning and constraints |
| --- | --- |
| topic | Topic（主题）, for example 审理方式, 补件期限. **Chinese source text; write in Chinese.** At most 100 characters; required on create, cannot be cleared. |
| content | Points（要点）: a faithful Chinese summary of the school's wording, plain text, at most 1000 characters. Translate English sources into Chinese; English and Traditional Chinese are generated by the backend, so do not submit translations. Required on create, cannot be cleared. |
| sort | Integer 0–100000; smaller values come first, equal values by id. Defaults to 0 on create. |
| sourceUrl / verifiedAt | Source page and check date, with the same rules as for calendar terms. |

Create with `school_asset_save(action=create, kind=application-rules)` and edit with action=update. Deleting a rule is available only on the CRM webpage. Term dates and deadlines belong in important-times, not in a rule's content; a rule may explain how a deadline works (for example rolling assessment) without restating the date.

## Videos: videos

bilibiliCode / youtubeCode are platform video identifiers; bilibiliTitle / youtubeTitle are their titles. null clears a field. The API does not fetch titles or parse arbitrary links for the agent; use explicit source information to identify the video code. After merging the patch with current values, at least one non-empty platform identifier must remain.

courseCodes is the complete list of associated course codes from this school. Omission preserves it; []/null removes associations while retaining the video record. The API validates ownership and deduplicates the list. It neither associates all school courses automatically nor uploads/deletes platform videos.

Example: “Make this scholarship apply to the whole school” changes range=all, clearing old course/subject associations. Do not also change qualification scope, location, or course tuition. See [saving rules](editing.md).
