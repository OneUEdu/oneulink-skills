# Courses and start dates

## Names and annual records

Use courses_list to locate a school's courses. Its year parameter validates the operating year; it does not filter the list by that year's fees or enrollment closure. course_get supplies the selected year's current values. The year must match both the user's evidence and crm_context.years; do not default to the current year.

| shared field | Meaning |
| --- | --- |
| courseEnName / courseCnName | English/Chinese course name（英文/中文名称）, shared across years. The English name cannot be cleared; null clears the Chinese name. |
| newVersion | Replacement course CRICOS code（替代课程）. Must identify another existing course at the same school, not itself. null removes the manual association; it does not close enrollment, delete the original course, or migrate data. The website prefers an existing manual target open for enrollment, otherwise it may fall back to automatic matching. Clearing this field does not guarantee the website stops recommending a replacement. |
| tuitionFeeViewMode | normal=default display（按默认显示）, week=weekly display（每周）. This changes presentation; stored tuition remains a total amount. |

shared and annual have independent revisions. Send only the revisions for modified groups. A shared-name change still requires an allowed year, but does not change that year's fees.

## Annual fees and student categories

All amounts below are non-negative integers in AUD; durations are non-negative whole weeks. null clears a recorded value; 0 is valid. The website may fall back to another fee category. An empty value or 0 does not guarantee an empty or free public quote; do not treat a fallback display as the selected year's recorded data.

| Webpage group | Total tuition / duration | Other fees |
| --- | --- | --- |
| Offshore international students（国际学生·境外） | totalTuitionFee / length | registrationFee registration（注册费）; materialFee materials（材料费）; internshipFee internship（实习费） |
| Onshore international students（国际学生·境内） | onshoreTuitionFee / onshoreLength | onshoreRegistrationFee, onshoreMaterialFee, onshoreInternshipFee |
| Domestic students（本地学生） | domesticTuitionFee / domesticLength | domesticRegistrationFee, domesticMaterialFee, domesticInternshipFee |
| Online study（线上授课） | onlineTuitionFee / onlineLength | onlineMaterialFee |
| VOE international students（国际学生·VOE） | voeTuitionFee / voeLength | voeMaterialFee |

The website displays VOE as a separate fee category labeled “VOE 课程”. Both its tuition and duration must be greater than 0 for it to appear in course-list and detail fee options. Entering tuition alone does not ensure display. Online study and VOE use separate fields.

Each group is stored independently, without automatic propagation or conversion. An onshore international student is not a domestic student; “onshore” alone does not justify writing domestic fields. Changing tuition does not automatically adjust duration or registration/material/internship fees. The Agent must explicitly save fee verification as described below; the backend does not infer it from an amount change.

For durations of at least 52 weeks, the public detail summary shows a rounded per-academic-year amount calculated as `total tuition / (weeks / 52)`; shorter courses show the total. The course list also lets visitors switch between total and per-academic-year amounts. Do not write a displayed annual amount back as CRM total tuition. The primary display follows the visitor's onshore/offshore selection, preferring the corresponding category before falling back to other groups with data.

For a supplied weekly rate, establish the category, billable weeks, and whether multiplying a constant weekly rate is appropriate. Do not enter a weekly rate as total tuition or infer duration from the course name. Fractional durations are unsupported; do not round them yourself.

## Fee verification when updating amounts

An authorized update involving course annual fee amounts defaults to precise/verified fees（精准学费／已核验学费）. Include `annual.isAccurateFee: true` in the same `course_update` patch as the changed amounts for that course and year, including when the existing flag is false or null. Apply this to tuition in every student category and to registration, materials and internship fees; 0 is a valid amount. This default is part of the authorized fee update and needs no separate confirmation. Only an explicit user instruction to keep an estimated/unverified or unset status overrides it; save false or null as requested.

Read back the saved amounts and `annual.isAccurateFee` with `course_get`. The completion summary must state the course, year, changed amounts and persisted precise/verified status, including any explicitly requested exception or failed verification-flag save. Do not report “verified” from the submitted patch alone. Apply the same rule to each fee-changing batch row and its result summary.

`isAccurateFee` is one course-year flag, not a separate flag for each fee category. A change only to duration, display mode or descriptive text does not invoke this default. Standalone discounts and packaged-course tools have their own contracts and do not expose this flag; do not invent a patch field or change an unrelated annual course record. Explain any verification status that the available tool cannot persist. The default does not permit guessing missing amounts, categories or years, or claiming that an official source was checked when it was not.

## Annual flags and supporting information

| annual field | Meaning and related behavior |
| --- | --- |
| isClose | Enrollment closed（停止招生）: true=closed for that year, false=open, null=unset. Does not delete the course or change dates. |
| isAccurateFee | Precise/verified fees（精准学费／费用已核验）: true=verified, false=unverified, null=unset. Authorized fee updates default to explicitly saving true under the rule above; the API does not set it automatically. The public international-student fee display shows a verification notice. Retain the actual source/user instruction in the edit reason without inventing official-source verification. |
| tuitionByScholarship | CRM label: 学费按奖学金显示. The public website displays “奖学金项目” (scholarship program) and a note that the corresponding tuition concession is fully provided by the school as a scholarship. This does not mean all tuition is waived. It neither associates a scholarship record nor calculates an amount. |
| domesticHasGovernmentSubsidy | CRM label: 本地学生政府补贴. The public website displays “政府补贴” (government subsidy) for domestic fees. The price still comes directly from domesticTuitionFee; the flag does not subtract a subsidy or establish individual eligibility. |
| courseInformation | Annual public course description（课程简介）, **English source text; write in English**: course overview, teaching content or learning outcomes, supporting existing rich-text HTML. Not fallback storage for unmapped locations, fees or feedback. Preserve unrelated existing content and images; see [destination selection](editing.md#choose-a-destination-before-writing). |
| courseUrl | Annual course source URL（课程来源网址）. The backend requires a valid http/https URL. |
| academicEntryRequirements | Annual academic entry requirements（学术入学要求）, **English source text; write in English**, plain text with line breaks, maximum 100,000 characters. Course-specific academic eligibility and selection conditions; follow the classification below. Omission preserves; null or whitespace clears. Does not change English requirements, fees or other years. |
| english | Annual English requirements note（英语要求说明）, **English source text; write in English**, plain text. Does not update scores automatically. |
| intakes | Deprecated intake note（旧入学说明）retained for historical readback only. No daily web editor or MCP write field; never reuse it for academic conditions. Actual dates use the existing structured date tools. Do not rewrite or migrate old content automatically. |
| ielts / pte | Annual entry-score requirements; see below. |

All four boolean flags accept null to unset them; null is distinct from false. Text and URL fields can be cleared with null. The API saves submitted fields. When a year has no recorded row, GET returns null; an edit may populate that annual record, but must not automatically copy the previous year.

## Academic entry requirements

Edit `annual.academicEntryRequirements` through `course_update` with the selected year's annual revision. Read the complete current text before replacing it and retain still-valid conditions. The source is English; faithfully translate supplied Chinese prose without inventing a qualification, score, assessment date or certainty. The API stores text; it does not calculate eligibility or guarantee admission. Do not copy a previous year automatically. `china_requirement` is a deprecated, read-only legacy field, not this field's source or fallback; never migrate its old values by inference.

Use this field for an institution's established course-specific academic entry and selection conditions:

- Required school completion, qualifications, degrees, relevant disciplines and prerequisite subjects.
- Grade thresholds such as GPA, WAM, average marks or ATAR, retaining the original scale, calculation basis, applicant group and applicable qualification.
- Portfolios, research proposals, auditions or academic interviews explicitly required for academic selection.
- Relevant professional experience only when the institution explicitly accepts/requires it as an entry qualification or alternative academic pathway.

Preserve separate admission pathways, AND/OR relationships, conditions, exceptions and applicable groups/intakes. A bare “65” does not establish 65%, a GPA conversion, a country-wide equivalence or a guaranteed offer. Clarify missing information only when it materially changes the authorized edit. Keep source links, known source dates and the basis for verification in the edit reason or linked feedback; the save timestamp is not a verification date.

IELTS/PTE, language exemptions and language-course pathways belong in the existing English fields. Dates, fees, discounts and course overviews use their own fields. Visa eligibility, health checks and placement compliance are not academic criteria; report a missing destination in OneULink CRM handling when no accurate field/tool exists. Internal advice belongs in consultationNote only when it independently fits that purpose. Unverified reports remain feedback/handling records rather than becoming established requirements.

This classification governs CRM resource editing, not feedback submission. Users may report any subject; lack of a writable destination never invalidates their feedback. OneUEdu feedback guidance only helps the Agent write a stand-alone report a consultant can understand without the chat. See [CRM feedback handling](feedback.md#choose-the-destination-after-verification).

Illustration only: if the institution explicitly establishes “a relevant bachelor degree with at least 65% average OR another bachelor degree plus two years of relevant experience”, retain both complete paths and the meaning of OR. Put an accompanying IELTS 6.5 requirement in English fields. A statement about a placement health check remains valid feedback even when no corresponding CRM editor exists. Do not save these hypothetical conditions into a real course.

## IELTS / PTE

listen=listening（听力）, speak=speaking（口语）, read=reading（阅读）, write=writing（写作）, overall=overall score（总分）. IELTS accepts 0–9 in 0.5 increments; PTE accepts integers from 0–90. The overall score is independent, not a sum or average of components. Do not convert between exams.

Objects merge individual components. To change only the overall requirement, send `ielts: {overall: 6.5}` and retain omitted components. A component set to null clears that component; ielts/pte set to null clears the whole group. An empty object does not clear a group. The API normalizes an all-empty PTE group to null.

## Cross-year start dates

course_dates_get and course_dates_update are separate from annual course_get/course_update records. manualDates includes manually recorded dates and the teachingStart of associated school intakes. selectedImportantTimeIds contains current associations; availableImportantTimes contains selectable intakes. officialDates/officialIntakes are read-only.

An update replaces both complete arrays, which must be submitted together. Preserve dates and associations not requested for removal. Include an intake's teachingStart when selecting it; remove that day's associations when removing the date. Multiple intakes can share a date: distinguish them by existing IDs, never guessed IDs. Two empty arrays clear only manual records and associations; they do not close enrollment.

effectiveDates combines manual and official dates strictly after the API's current date. If none are available, it falls back to the school calendar with fallbackSource=school. Past and same-day dates may remain in the source while absent from effectiveDates. Editing an associated calendar can also change course dates and their revision.

Example: “Close enrollment next year” changes isClose for the established year. “Remove one manual start date” changes the complete date set and corresponding associations while retaining official dates. These operations are not interchangeable.

## VOE and price entry

VOE commonly means Verification of Enrolment（在读/注册证明）. The [PIA glossary](https://pia.edu.au/about-us/resources-support/glossary/) gives this expansion; [PAX admission information](https://www.pax.edu.au/future-students/) describes a process issuing VOE after acceptance for international students on non-student visas. These are institutional examples, not universal visa-eligibility rules.

In this system, VOE is a separate international-student fee group: voeTuitionFee is that category's total course tuition, voeLength is duration in weeks, and voeMaterialFee is the materials fee. It is neither online study nor an administrative fee for issuing a VOE document. Do not enter a separately quoted document-issuance fee as voeTuitionFee. Use an institution's explicit VOE-category quote; do not classify automatically from a visa name or the word “online”.

The following price behavior is established by source code; do not ask the user whether the system deducts these amounts automatically:

- Domestic pricing takes domesticTuitionFee directly. domesticHasGovernmentSubsidy adds a label without subtracting another subsidy. To display a particular subsidized quote, that amount must itself be entered in this field; entering the original price and toggling the flag will not calculate it.
- tuitionByScholarship adds only the scholarship-program label and explanation. Without a separate discount record, the website uses the corresponding tuition field; the flag does not discount or waive it. If only the flag change is requested, leave amounts unchanged.
- With a separate course discount, the course card uses totalTuitionFee as the original price and discountedFee directly as the discounted price. Their difference/ratio drives the savings display. Do not enter a deduction amount as discountedFee, overwrite the original tuition again, or deduct twice. See [discounts](discounts.md).

Ask for missing details only when the institution's specific evidence does not establish the target category, total price, or necessary billing basis. Do not turn the known display mechanics into unresolved business questions.

See [saving and conflicts](editing.md).

## Consultation notes（课程咨询提示）

Edit `shared.consultationNote` with `course_update` and the shared revision. This optional plain-text field (maximum 5,000 characters) applies across years; the tool still requires an allowed year, but note edits do not alter annual fees, intakes or `isClose`. `shared.consultationNoteUpdatedAt` is read-only UTC metadata. Omission preserves the note; null or whitespace clears it. Replacement is complete, so retain other still-valid advice.

Use it for verified course-specific context affecting internal advice or applications, such as a registered course currently lacking enough enrollment to run. A missing public field/tool alone does not justify a note, and a note does not complete the missing public update. Follow [self-contained note writing](editing.md#write-self-contained-consultation-notes) when adding or supplementing it. State the applicable dates or conditions; do not infer cancellation, closed registration, or school-wide unsuitability. Put school-wide facts in the school's consultationNote.

These notes stay inside OneULink under existing course read/edit permissions. Read the exact CRM course detail and, when relevant, the school's note. Neither OneUEdu MCP nor public website pages return them, even for connected OneUEdu accounts. Do not copy internal advice into public fields or external replies without authorization. Avoid credentials or unrelated personal information. Follow [feedback classification](feedback.md) before turning a report into internal advice. A submitted report alone is not verification, and a recorded update time is not evidence that a statement remains current.

## Packaged course feedback

`course_package` is text-only feedback; it need not originate from a file. Read the original and customer follow-ups, then verify school/course codes and use `course_packages_list` to find the existing combination. Use `course_package_get` before `course_package_update`, or `course_package_create` for a new combination. These tools require resource read plus the corresponding existing package create/update permission; missing tools do not authorise writing package prices into a single course or consultation note.

One combination stays one package. `totalTuitionFee` is the offshore package total, including all historic values; `onshoreTotalTuitionFee` is separate. Missing onshore means no quote, never offshore fallback. Update only the stated side; omit the other side to preserve it. If a quote explicitly applies to both, set both prices to the same amount. Duration and material fees are shared, and course order is preserved. Do not divide a package price between course records or overwrite single-course fees. Pass feedbackIds to associate the edit; use the resource revision and a stable idempotencyKey. Read back prices and the website/sync outcome, then update the feedback with actual work and remaining gaps. Saving a package does not resolve feedback.
