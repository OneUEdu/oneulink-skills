# Courses and start dates

## Names and annual records

Use courses_list to locate a school's courses. Its year parameter validates the operating year; it does not filter the list by that year's fees or enrollment closure. course_get supplies the selected year's current values. The year must match both the user's evidence and crm_context.years; do not default to the current year. The year is the commencement year the annual record applies to: when a price list is split by start date, each start date's fees belong to that date's year. When a price list covers only part of a year (for example intakes up to 30 June 2027), no field stores the cutoff. Save the fees for that year, state the cutoff in the edit reason and report it as an open item; do not put it in courseInformation or consultationNote.

| shared field | Meaning |
| --- | --- |
| courseEnName / courseCnName | English/Chinese course name（英文/中文名称）, shared across years. The English name cannot be cleared; null clears the Chinese name. |
| newVersion | Replacement course CRICOS code（替代课程）. Must identify another existing course at the same school, not itself. null removes the manual association; it does not close enrollment, delete the original course, or migrate data. The website prefers an existing manual target open for enrollment, otherwise it may fall back to automatic matching. Clearing this field does not guarantee the website stops recommending a replacement. Set it on the old course, pointing to the replacement. Until the old course is closed for the relevant years (annual.isClose=true), the website and public MCP only show that a new code is registered; after closure they show the old course as replaced. Keep fees and start dates on the new course, and update packages that still list the old code. |
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

### Domestic tuition by study mode

Record domestic online total tuition in `annual.domesticTuitionDetails.online.totalTuitionFee` as whole-course AUD, even when no duration is known. The legacy `annual.onlineTuitionFee` is a separate category. Keep the existing domestic amount in `annual.domesticTuitionFee`. Set `annual.domesticTuitionDetails.onCampus: true` only if the school confirms this amount applies to on-campus study.

Record English conditions in `onCampusCondition` and/or `online.condition` for the applicable mode. For early-cohort or limited-intake rates, retain the eligibility and intake window together with the amount. The website and public fee tool display the applicable condition with the rate. Do not infer exact intake dates, treat a conditional rate as a standard quote, or apply it across years without evidence.

Read the requested year first. Updating `domesticTuitionDetails` replaces the object, so preserve unaffected study modes and conditions. Omit the object for ordinary edits, or pass `null` to clear it. When recording a domestic study-mode amount or condition, leave the shared `isAccurateFee` unchanged unless the entire annual fee record is being verified.

The website displays VOE as a separate fee category labeled “VOE 课程”. Both its tuition and duration must be greater than 0 for it to appear in course-list and detail fee options. Entering tuition alone does not ensure display. Online study and VOE use separate fields.

Each group is stored independently, without automatic propagation or conversion. An onshore international student is not a domestic student; “onshore” alone does not justify writing domestic fields. Changing tuition does not automatically adjust duration or registration/material/internship fees. The Agent must explicitly save fee verification as described below; the backend does not infer it from an amount change.

For durations of at least 52 weeks, the public detail summary shows a rounded per-academic-year amount calculated as `total tuition / (weeks / 52)`; shorter courses show the total. The course list also lets visitors switch between total and per-academic-year amounts. Do not write a displayed annual amount back as CRM total tuition. The primary display follows the visitor's onshore/offshore selection, preferring the corresponding category before falling back to other groups with data.

For a supplied weekly rate, establish the category, billable weeks, and whether multiplying a constant weekly rate is appropriate. Do not enter a weekly rate as total tuition or infer duration from the course name. Fractional durations are unsupported; do not round them yourself.

Material fees in every group are the whole-course total, not a yearly amount. When the institution charges materials per period (for example A$350 every six months), save the total the course incurs at its quoted duration and state the basis in the edit reason; do not save only the first instalment. A course-level enrolment fee belongs in the group's registration fee; a school-wide application fee belongs in `school_update.applicationFee`. When one price applies to onshore and offshore students, fill both groups explicitly. Duration is the number of weeks the institution quotes for this way of studying. If the institution only admits students with credit transfer and quotes the reduced weeks (for example 31 weeks while CRICOS registers 72), enter the quoted weeks and record the prerequisite in academicEntryRequirements. Weeks inside a package belong to the [package tools](#packaged-course-feedback), not to the single course.

## Fee verification when updating amounts

For ordinary annual fee amount updates, include `annual.isAccurateFee: true` in the same `course_update` patch as the changed amounts for that course and year. This covers tuition, registration, materials and internship fees, including zero. If the user requests an estimated or unset status, use false or null as requested. When recording domestic tuition by study mode, including a corresponding correction to the campus amount, preserve the existing course-year flag unless the full annual record is being verified.

Read back the saved amounts and `annual.isAccurateFee` with `course_get`. The completion summary must state the course, year, changed amounts and persisted precise/verified status, including any explicitly requested exception or failed verification-flag save. Do not report “verified” from the submitted patch alone. Apply the same rule to each fee-changing batch row and its result summary.

`isAccurateFee` is one course-year flag, not a separate flag for each fee category. A change only to duration, display mode or descriptive text does not invoke this default. Standalone discounts and packaged-course tools have their own contracts and do not expose this flag; do not invent a patch field or change an unrelated annual course record. Explain any verification status that the available tool cannot persist. The default does not permit guessing missing amounts, categories or years, or claiming that an official source was checked when it was not.

## Annual flags and supporting information

| annual field | Meaning and related behavior |
| --- | --- |
| isClose | Enrollment closed（停止招生）: true=closed for that year, false=open, null=unset. Does not delete the course or change dates. |
| isAccurateFee | Precise/verified fees（精准学费／费用已核验）: true=verified, false=unverified, null=unset. For ordinary fees and the domestic study-mode exception, follow the rule above. The public international-student fee display shows a verification notice. Retain the actual source and user instruction in the edit reason. |
| tuitionByScholarship | CRM label: 学费按奖学金显示. The public website displays “奖学金项目” (scholarship program) and a note that the corresponding tuition concession is fully provided by the school as a scholarship. This does not mean all tuition is waived. It neither associates a scholarship record nor calculates an amount. |
| domesticHasGovernmentSubsidy | CRM label: 本地学生政府补贴. The public website displays “政府补贴” (government subsidy) for domestic fees. The price still comes directly from domesticTuitionFee; the flag does not subtract a subsidy or establish individual eligibility. |
| courseInformation | Annual public course description（课程简介）, **English source text; write in English**: course overview, teaching content or learning outcomes, supporting existing rich-text HTML. Not fallback storage for unmapped locations, fees or feedback. Preserve unrelated existing content and images; see [destination selection](editing.md#choose-a-destination-before-writing). |
| courseUrl | Annual course source URL（课程来源网址）. The backend requires a valid http/https URL. |
| academicEntryRequirements | Annual academic entry requirements（学术入学要求）, **English source text; write in English**, plain text with line breaks, maximum 100,000 characters. Course-specific academic eligibility and selection conditions; follow the classification below. Omission preserves; null or whitespace clears. Does not change English requirements, fees or other years. |
| english | Annual English requirements note（英语要求说明）, **English source text; write in English**, plain text. Preserve applicability, exemptions and alternatives here. Verified IELTS/PTE thresholds also require the corresponding structured scores; this note does not populate them. |
| researchDegree | Annual structured research-degree details（研究学位资料）for doctoral and research master's courses, **English source text; write in English**. Whole-field replacement; see [research degree details](#research-degree-details). |
| intakes | Deprecated intake note（旧入学说明）retained for historical readback only. No daily web editor or MCP write field; never reuse it for academic conditions. Actual dates use the existing structured date tools. Do not rewrite or migrate old content automatically. |
| ielts / pte | Annual entry-score requirements; see below. |

All four boolean flags accept null to unset them; null is distinct from false. Text and URL fields can be cleared with null. The API saves submitted fields. When a year has no recorded row, GET returns null; an edit may populate that annual record, but must not automatically copy the previous year.

## Teaching arrangements and practical sites

`annual.teachingArrangements` stores the selected year's class schedule and practical (workshop) locations, for the course in general and optionally per course campus. A practical site that is not a campus record, such as a commercial kitchen or automotive workshop, is a `workshopLocations` entry `{name, address}`: an optional site name and the full street address with suburb, state and postcode. It is shown as course information; it does not create a campus or change enrolment-campus relationships. Use `{campusId}` only to reference an existing campus of the school.

A campus entry that omits workshopLocations inherits the general entry; `[]` explicitly lists none for that campus. The field is replaced as a whole, so read the current value and resubmit the entries to keep. It is stored per year and not inherited: when the same sites apply to several years, write each year.

## Academic entry requirements

Edit `annual.academicEntryRequirements` through `course_update` with the selected year's annual revision. Read the complete current text before replacing it and retain still-valid conditions. The source is English; faithfully translate supplied Chinese prose without inventing a qualification, score, assessment date or certainty. The API stores text; it does not calculate eligibility or guarantee admission. Do not copy a previous year automatically. `china_requirement` is a deprecated, read-only legacy field, not this field's source or fallback; never migrate its old values by inference.

Use this field for an institution's established course-specific academic entry and selection conditions:

- Required school completion, qualifications, degrees, relevant disciplines and prerequisite subjects.
- Prerequisite courses the institution imposes, including ones completed at another provider (for example “students must complete SIT30821 Certificate III before this Certificate IV”). A training.gov.au statement that a qualification has no entry requirements describes only the national qualification; it does not override the institution's own rule.
- Grade thresholds such as GPA, WAM, average marks or ATAR, retaining the original scale, calculation basis, applicant group and applicable qualification.
- Portfolios, research proposals, auditions or academic interviews explicitly required for academic selection. For doctoral and research master's courses, also structure these in `researchDegree` (see [research degree details](#research-degree-details)).
- Relevant professional experience only when the institution explicitly accepts/requires it as an entry qualification or alternative academic pathway.

Preserve separate admission pathways, AND/OR relationships, conditions, exceptions and applicable groups/intakes. A bare “65” does not establish 65%, a GPA conversion, a country-wide equivalence or a guaranteed offer. Clarify missing information only when it materially changes the authorized edit. Keep source links, known source dates and the basis for verification in the edit reason or linked feedback; the save timestamp is not a verification date.

IELTS/PTE, language exemptions and language-course pathways belong in the existing English fields. Dates, fees, discounts and course overviews use their own fields. Visa eligibility, health checks and placement compliance are not academic criteria; report a missing destination in OneULink CRM handling when no accurate field/tool exists. Internal advice belongs in consultationNote only when it independently fits that purpose. Unverified reports remain feedback/handling records rather than becoming established requirements.

This classification governs CRM resource editing, not feedback submission. Users may report any subject; lack of a writable destination never invalidates their feedback. OneUEdu feedback guidance only helps the Agent write a stand-alone report a consultant can understand without the chat. See [CRM feedback handling](feedback.md#choose-the-destination-after-verification).

Illustration only: if the institution explicitly establishes “a relevant bachelor degree with at least 65% average OR another bachelor degree plus two years of relevant experience”, retain both complete paths and the meaning of OR. Put an accompanying IELTS 6.5 requirement in English fields. A statement about a placement health check remains valid feedback even when no corresponding CRM editor exists. Do not save these hypothetical conditions into a real course.

## Research degree details

`annual.researchDegree` holds structured details for doctoral and research master's courses（博士 / 研究型硕士）: degree type, supervisor policy, entry routes, research application documents, interview, steps, a notice, course structure and application deadlines. Edit it through `course_update` with the selected year's annual revision. Omission preserves the field, null clears it, and an object replaces the whole field: read the current value with `course_get` first and keep every still-valid part. Every key is optional; record only what the institution states. Text is English source; faithfully translate supplied Chinese. Empty text, empty lists and an empty object are removed by the backend. The tool schema states the limits and rejects unknown keys.

Classify each fact before writing:

- **Academic, grade and experience conditions → `entryRoutes`.** One route per alternative way in; routes are OR, conditions inside a route are AND. Write each condition as a complete sentence and keep the institution's scale (“Average of at least 65%”, “WAM 70”). Also keep the full prose in `academicEntryRequirements`; the routes do not replace it. Do not split one pathway into several routes unless the source states the alternatives. Put conditions shared by every route (research documents, interview, no guaranteed admission) in `entryRoutesNote`.
- **English, fees and start dates stay in their existing fields** (`english`/`ielts`/`pte`, fee fields, `course_dates_update`). A deadline's `intakeDate` only labels which intake the deadline belongs to.
- **Never infer a supervisor policy.** Choose `required_before_application`（申请前必须取得导师同意）, `contact_recommended`（建议先联系导师）or `assigned_by_provider`（学校按研究计划分配）only when the institution says so; when it is silent use `not_stated`（学校没写明）. A supervisor `note` without a `policy` is rejected.
- **Never infer numbers.** A bare “65” is not 65%, a GPA or a WAM. Record `maxWords` only for a stated word limit, credits only as published (up to two decimals; when total, coursework and research are all given, coursework + research must equal the total). Do not derive credits from unit counts or percentages.
- **`structure.progressionRule` is a post-enrolment progression condition**（升读条件）, for example a first-year average of 70% to continue into the doctoral stage. It is not an entry requirement; entry conditions go in `entryRoutes`.
- **Deadlines:** use `closesOn` (YYYY-MM-DD, optionally `closesAt` HH:MM and an IANA `timezone` such as `Australia/Melbourne`) for a stated date. Use `weeksBeforeStart` with `intakeDate` only when the institution states “at least N weeks before the start date”; do not convert it into a date or a date into weeks. Each deadline has exactly one of the two. `audience` is `all`, `offshore` or `onshore`.
- **Documents** list research-specific materials (research proposal, motivation statement, writing sample, referees). `requirement` is `required`（必交）or `conditional`（视情况）only as stated; links must be https official templates or guidelines. General admission documents and English test results are not research documents.
- `interview` is `required`, `possible` or `not_stated`; `steps` lists up to eight stated application steps in order; `notice` is one caution the website shows in a highlighted box.

Read back with `course_get` and report the course, year and saved research-degree parts. Leave unstated parts empty and report them as not provided by the institution rather than filling them in.

## IELTS / PTE

When authorized to update a course's English requirements, read `annual.english`, `annual.ielts` and `annual.pte` together for the exact course and year. Map every verified numeric threshold to its structured field, and preserve its applicant scope, conditions, exemptions and alternatives in the English note. Submit the necessary note and score changes in the same `course_update.patch.annual`; prose is not a substitute for supported structured values. Already-correct scores need no redundant write, but still require readback verification.

listen=listening（听力）, speak=speaking（口语）, read=reading（阅读）, write=writing（写作）, overall=overall score（总分）. IELTS accepts 0–9 in 0.5 increments; PTE accepts integers from 0–90. The overall score is independent, not a sum or average of components. Do not convert between exams.

For example, verified course evidence stating “IELTS 7.0 overall, no component below 6.5” requires all five IELTS fields:

```json
{
  "annual": {
    "english": "IELTS 7.0 overall, with no component below 6.5.",
    "ielts": {"overall": 7, "listen": 6.5, "speak": 6.5, "read": 6.5, "write": 6.5}
  }
}
```

This is a patch illustration, not evidence about any real course. Retain the actual source's conditions in `english`. “Each component” establishes all four minima; a stated overall alone establishes no component score. Apply the same mapping to explicitly supplied PTE thresholds, never by converting IELTS. An English-medium qualification exemption or a warning that other awards have different requirements does not by itself invalidate the verified thresholds for this course's test-score pathway. Conversely, unresolved course/year applicability or conflicting thresholds for different applicant groups/intakes cannot be reduced to one guessed score set: retain the qualified explanation and record any required structured update as unfinished. A note-only change about exemptions or alternatives with no established numeric change may leave scores untouched; absence of an exam in the source does not establish that its recorded scores should be cleared.

Objects merge individual components. To change only the overall requirement, send `ielts: {overall: 6.5}` and retain omitted components. A component set to null clears that component; ielts/pte set to null clears the whole group. An empty object does not clear a group. The API normalizes an all-empty PTE group to null.

After saving, call `course_get` for the same school, course and year and compare the note and every supported score with the verified source, including already-correct values. If a required value is missing, stale or unsaved, the score update remains unfinished. When handling feedback, do not resolve/close it merely because the note saved; record the gap and retain pending/in_progress until the necessary correction is verified.

## Cross-year start dates

course_dates_get and course_dates_update are separate from annual course_get/course_update records. manualDates includes manually recorded dates and the teachingStart of associated school intakes. selectedImportantTimeIds contains current associations; availableImportantTimes contains selectable intakes. officialDates/officialIntakes are read-only. manualDates are course commencement dates, not orientation, census or application deadlines. The fees for each date come from the course_update annual record of that date's year.

An update replaces both complete arrays, which must be submitted together. Preserve dates and associations not requested for removal. Include an intake's teachingStart when selecting it; remove that day's associations when removing the date. Multiple intakes can share a date: distinguish them by existing IDs, never guessed IDs. Two empty arrays clear only manual records and associations; they do not close enrollment.

effectiveDates combines manual and official dates strictly after the API's current date. If none are available, it falls back to the school calendar with fallbackSource=school. Past and same-day dates may remain in the source while absent from effectiveDates. Editing an associated calendar can also change course dates and their revision.

Each manual start date may carry one optional end date（结课日）: `manualEndDates` is `[{startDate, endDate}]`, keyed by a date in manualDates, including dates kept as calendar associations. Enter an end date only when the institution's evidence states it for that start date (for example a trimester table "01 March – 16 July" gives startDate 2027-03-01, endDate 2027-07-16; a mid intake that finishes with the same trimester gets that trimester's end). Never derive it from the course length, a school-calendar teachingEnd or officialIntakes. Omitting manualEndDates keeps the end dates of start dates that remain; sending it replaces all end dates, so read the current list and resubmit the ones to keep. Removing a start date removes its end date. The end date cannot precede its start date. Year-end or semester-end dates that are not tied to a course start belong in the school calendar（important-times teachingEnd）, not here.

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

Each course in a package may carry its own `length` and `tuitionFee` as the institution quotes them for that package, often after credit transfer (for example 31 weeks in the package while CRICOS registers 72). Enter the quoted values; do not replace them with CRICOS or standalone durations. A course's `tuitionFee` is the offshore amount; there is no onshore per-course field. These values do not change the course's own annual fees. Package totals (`totalLength`, `totalTuitionFee`, `onshoreTotalTuitionFee`) are the institution's quoted figures; do not compute them by summing courses.

When an update includes `courses`, send the complete ordered list and repeat every course's `length` and `tuitionFee`; values left out are cleared. Packages have no year, so a saved price applies to all intakes. If a quote covers only some intakes, state that in the reason and report it.
