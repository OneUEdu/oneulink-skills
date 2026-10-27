# School assets

Locate records with school_assets_list by kind, read current values and revision with school_asset_get, and edit existing records with school_asset_update. kind values are campuses=campuses（校区）, important-times=academic calendars（学校校历）, scholarships=scholarships（奖学金）, and videos=videos（视频）. Use an id from the same school's list of that kind; video IDs are opaque strings.

## Campuses: campuses

| Field | Meaning and related behavior |
| --- | --- |
| campusName | Campus name（校区名称）. If changed without displayCampusName, the API also updates the display name. |
| displayCampusName | Separate display name（显示名称）. To rename a campus while retaining its old display name, explicitly submit that old display name too. |
| state | Australian state/territory abbreviation. Use the eight values in the current schema. |
| city / postcode | City（城市）/ four-character postcode（邮编）. Preserve leading zeroes, for example 0820. |
| addressLine1–4 | Four independent address lines（地址行）. Omission preserves each line; null clears that line. |

All these fields can be cleared with null when editing. Missing source information is not an instruction to clear them. The API rebuilds detailAddress from the address lines, city, state, and postcode and marks the record as manually modified. rawState/rawCampusName/rawDetailAddress/locationKey/isModify/detailAddress are read-only; do not edit source values. The webpage's restore-original-data action is not exposed through this MCP.

## Academic calendars: important-times

| Field | Meaning and constraints |
| --- | --- |
| year / intake | Calendar year / intake name（年份/学期名称）. An intake name must be unique within the same school and year. Calendar years are independent of the course-fee year allowlist. |
| teachingStart / teachingEnd | Teaching start/end（教学开始/教学结束）. Required on the webpage; MCP does not allow clearing them. End cannot precede start. |
| censusDate | The school's published Census Date. Follow the school's definition; do not treat it as an application deadline. |
| examsStart / examsEnd | Exam start/end（考试开始/考试结束）. End cannot precede start. |
| resultRelease | Results release date（成绩公布）. |
| applicationClose | Application deadline（申请截止日期）. Cannot be after teachingStart. |

Use YYYY-MM-DD dates. year accepts 1900–2200; every non-empty date must fall within year−1 through year+1. Dates other than teaching start/end may be cleared with null; the intake name cannot be cleared. Calendar changes can affect associated course dates and revision, not just this row's display.

## Scholarships: scholarships

| Field | Meaning and related behavior |
| --- | --- |
| name / description | Name / eligibility or usage conditions（奖学金名称/使用条件）. Both are required plain text. description is not course information or a calculation formula. |
| level | Qualification scope（学历范围）: all=all（全部）, diploma=diploma（专科）, bachelor=bachelor's（本科）, master=master's（研究生）. Independent of range. |
| range | Applicability（适用范围）: all=whole school（全部）, subject=subject category（专业）, course=specific course（课程）. |
| subject | Required for range=subject. Two-character classification code; preserve leading zeroes. The webpage calls 00 “全校”, but the public site matches subject code 00 exactly, not as a whole-school wildcard. Use range=all for the whole school. Use the schema for other code meanings; do not guess. |
| courseCode | Required for range=course and must belong to this school. |
| studentLocation | Student location（学生所在地）: onshore=in Australia（境内）, offshore=outside Australia（境外）, all=anywhere（全部）. This is not citizenship. |
| endDate | Deadline（截止日期）. null clears it; a missing or expired date produces websiteVisible=false. |

Changing range clears inapplicable subject/courseCode fields and rebuilds associations. Submitting subject/courseCode alone does not switch range. Do not copy inapplicable fields from a read result back into patch. Scholarship records do not automatically change course tuition or tuitionByScholarship.

websiteVisible and visibilityReason are read-only derived information: missing_end_date means no date; expired means the deadline has passed. Comparison uses the API's current date, and the deadline day still satisfies the date condition. This checks only dates; the public website also filters by course qualification, applicability, and visitor location. It does not prove website synchronization or imply an available publish operation.

## Videos: videos

bilibiliCode / youtubeCode are platform video identifiers; bilibiliTitle / youtubeTitle are their titles. null clears a field. The API does not fetch titles or parse arbitrary links for the agent; use explicit source information to identify the video code. After merging the patch with current values, at least one non-empty platform identifier must remain.

courseCodes is the complete list of associated course codes from this school. Omission preserves it; []/null removes associations while retaining the video record. The API validates ownership and deduplicates the list. It neither associates all school courses automatically nor uploads/deletes platform videos.

Example: “Make this scholarship apply to the whole school” changes range=all, clearing old course/subject associations. Do not also change qualification scope, location, or course tuition. See [saving rules](editing.md).
