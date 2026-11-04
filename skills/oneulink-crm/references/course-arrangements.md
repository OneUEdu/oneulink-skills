# Course payment, teaching and placement

Read this for payment schedules/deposits（付款清单／首付款）, class schedules（上课时间）, workshop locations（实操地点）or placement hours（实习工时）. Use the tools actually discovered from the current connection: an installed Skill does not upgrade the MCP or backend. If the fields below are absent, report the missing capability; do not send unsupported keys or put the facts in courseInformation/consultationNote instead. Deployment or client discovery may need updating by the normal release process.

Before an authorized edit, establish the exact school/course, evidence year and student category, then call course_get for the complete current values, revisions and campuses. Use an allowed year from crm_context.years that matches the evidence. A personal offer can establish the amounts for that applicant without establishing a generally applicable course price. Confirm the reusable scope; do not publish personal scholarships, payments already made, student names or individual due dates as course defaults. A request to analyze an attachment alone does not authorize editing CRM records.

## Optional payment plans

| Student category | annual field |
| --- | --- |
| Offshore international | offshorePaymentPlan |
| Onshore international | onshorePaymentPlan |
| Domestic | domesticPaymentPlan |
| Online | onlinePaymentPlan |
| VOE | voePaymentPlan |

Each field is independent for the exact selected year and category. Payment plans have no campus association. Omission preserves a plan, null clears it, and an object replaces the **whole plan**, including payments/items/note. Read and merge with the current plan, retaining unrelated valid rows and conditions; do not send a one-row patch that accidentally deletes later installments. Do not copy another year/category into an empty plan.

A plan contains payments in display order and an optional note for actual payment conditions. Each payment has a required English label, optional English dueText, optional isInitial, and items. isInitial=true identifies an explicitly established upfront payment/deposit, such as a payment required to accept an offer. It does not add another charge, create a separate deposit amount, establish refundability, or follow automatically from array position. Only record confirmed rows; a source showing the upfront payment does not require inventing the rest of the schedule.

Items use type=tuition/registration/material/internship/coe/combined/other. Use the source's actual fee meaning: an Application Fee is not automatically a registration fee, and a Digital Access Fee is not automatically materials. Use other with the original English name where no enum matches; combined also requires a name. amountMinor is **AUD integer cents**: AUD 9,800 = 980000. This differs from the legacy numeric tuition/fee fields, which use integer AUD. Zero means explicitly free; null means not provided. Unknown amounts are not zero.

unit defaults to payment (a one-time amount); alternatives are month/week/subject/credit, where amountMinor is the unit price. quantity may be omitted/null when unknown; known quantities are positive, month/week/subject use integers and credit permits up to four decimal places. A credit quantity is not the course duration. For a known item, multiply exactly and round HALF_UP to cents; a unit price without its quantity is not a known installment total. Follow the current schema for safe money, text and row limits.

Preserve component fees once. An offer's bold total is a check against its listed components, not a fourth item. Do not enter both a combined total and its components as payable items. If an upfront tuition payment is part of a term's tuition, represent the later **balance**, only when the evidence establishes that balance. Do not repeat the whole term tuition after the deposit or subtract an unverified amount. Keep an actual inclusion rule in note when supplied.

Example, only after the source establishes an applicable course/year/category and reuse is authorized: an offer lists upfront tuition AUD 9,800, application AUD 250 and digital access AUD 100, with AUD 10,150 required on acceptance. The selected plan can contain this one confirmed payment:

```json
{
  "payments": [
    {
      "label": "Offer acceptance payment",
      "dueText": "On offer acceptance",
      "isInitial": true,
      "items": [
        { "type": "tuition", "amountMinor": 980000 },
        { "type": "other", "name": "Application Fee", "amountMinor": 25000 },
        { "type": "other", "name": "Digital Access Fee", "amountMinor": 10000 }
      ]
    }
  ]
}
```

The item sum is 1015000 cents = AUD 10,150. Do not store that sum again or overwrite totalTuitionFee with it: upfront tuition does not establish total course tuition. If only an undivided upfront amount is established, one named combined item may express it; do not invent its breakdown. An unknown upfront amount has no numeric summary, and the first unmarked row is not evidence of a deposit. Do not add version, complete, isEstimate, currency, totals or campus fields. This example contains no real course identifier/year and is not an instruction to change records.

Payment labels, dueText, custom item names and note are English source text; translate supplied prose faithfully while preserving official names and conditions. Do not add disclaimers or personal offer terms as generic payment conditions. CRM persistence is separate from website translation/publication; current website rows show the initial-payment badge, not a separate upfront-price summary card.

## Optional annual teaching arrangements

Use annual.teachingArrangements with the annual revision. Its optional general applies to the course, and optional campuses contains only differing arrangements for specific course campuses. Class schedules and workshop lists are optional at both levels; neither is mandatory. Timetables/workshop locations do not set workComponent or prove an internship exists.

Each arrangement may contain classSchedule (English text) and workshopLocations. Resolve the two fields separately: missing/null/blank classSchedule inherits the default; missing/null workshopLocations inherits it, while [] explicitly lists no workshops and blocks location inheritance. An empty default object means no recorded default, not cancellation. Store raw overrides only, not a flattened effective preview.

Choose override campusId from course_get.campuses with isCourseCampus=true. A referenced workshop uses {campusId} for any active location at the same school, even when not a course enrollment campus. These are the returned campus record IDs, not location_key, CRICOS codes or guessed IDs. If no suitable campus record exists, a workshop may instead use {name?,address} with its complete, required official address; this records a workshop, not a new campus or enrollment link. Preserve addresses literally, and do not mix campusId with copied name/address. If campus choices are not returned, do not infer that the course has no campuses.

Illustration only; replace IDs using current course_get records before an authorized write:

```json
{
  "general": {
    "classSchedule": "Monday and Wednesday, 09:00–17:00",
    "workshopLocations": [{ "campusId": 4 }]
  },
  "campuses": [
    { "campusId": 2, "classSchedule": "Tuesday and Thursday, 10:00–18:00" },
    { "campusId": 3, "workshopLocations": [] }
  ]
}
```

Here campus 2 inherits the workshop; campus 3 inherits the default timetable and explicitly lists no workshops. A new object replaces the whole teachingArrangements field, so preserve other valid defaults/overrides and omit GET-enriched campusName and resolved workshop name/address for campus references. Creating campus master records or enrollment associations remains unavailable through these fields; see [school assets](assets.md#campuses-campuses).

## Existing shared placement fields

Use shared.workComponent=Yes/No/null for included/not included/not recorded; the API values are case-sensitive. shared.workComponentHoursPerWeek, shared.workComponentWeeks and shared.workComponentHours are optional positive whole integers or null. They are existing cross-year course fields, not fields inside teachingArrangements. Record only hours/weeks established by the source; do not derive missing hours by assuming an intensive placement schedule. Changing workComponent to No/null requires explicitly clearing any recorded positive hour/week fields in the same shared patch. Do not introduce internship_notes or treat a placement fee/workshop address as proof of work placement.

Use the shared revision for these fields, the annual revision for plans/teaching, and both when changing both groups. An allowed year is still required for a shared-only edit. Read back each intended field and report its saved scope separately from pending website synchronization; follow [saving and conflicts](editing.md).
