# Course discounts

Use course_discounts_list, course_discount_get, and course_discount_update for existing discounts; schoolCode is required. Lists filter courseCode/type/location exactly and include future and expired records. A successful query does not imply a discount is currently applicable. Obtain id from the list; discountCode is not the detail identifier.

## Amounts and target course

| Field | Meaning and behavior |
| --- | --- |
| courseCode | Existing course code at this school. Changing it reassociates the discount; it does not edit the course master record or annual fees. |
| type | reduction=fixed reduction（固定减免）, discount=percentage discount（打折扣）. Changing type does not calculate an amount. |
| discountedFee | Final discounted fee（优惠后费用）in AUD, non-negative with at most two decimal places. Not a deduction amount or percentage. Accepts a number or decimal string; reads return a two-decimal string. |
| description / enDescription | Both take English prose. `description` is the English source used by the v2 translation model, despite its generic CRM label（优惠说明）; `enDescription` is the separate CRM English-description field（英文说明）. Do not assume `description` is Chinese or automatically copy a change into both fields. Plain text; null clears them. |
| location | Applicable student location（学生所在地）: onshore=in Australia（境内）, offshore=outside Australia（境外）. Does not establish citizenship or domestic-student status. |
| haveStudentVisa | Student visa required（要求持有学生签证）: true=required, false=not required, null=unset. This is an eligibility condition, not an update to an individual student's visa status. |

discountCode, the raw discount field, courseEnName, and timezone are read-only. The backend preserves and validates raw discount, which neither the webpage nor MCP exposes for editing. Report validation failures caused by legacy data rather than bypassing them. Saving a discount does not automatically change course totalTuitionFee.

## Citizenship and admission conditions

citizenship is the complete list of eligible citizenships（适用国籍）. Omission preserves it; []/null means unrestricted. Use CRM dictionary short codes, usually three uppercase characters such as CHN/AUS/VNM, with D retained for Germany, or complete webpage values such as CHINA - CHN. This is not a strict ISO dictionary; legacy options such as TMP/RKS remain valid. Do not expand the full catalog or guess a standard code. The API checks actual dictionary entries and stores full webpage options; it rejects two-letter codes, lowercase codes, unknown codes, and country names alone.

In invalid_country_code, citizenship.N identifies patch.citizenship[N]. Verify and correct the indicated value. A failed request does not save other fields. When citizenship is not being edited, leave legacy invalid values untouched rather than copying them into a new patch.

admissionYear is the complete admission-period list（入学时间）: `{type:"year",value:"2027"}` means an academic year; `{type:"intake",value:"2027-02"}` means an intake month. Year strings range from 1900–2199, independently of the course-year allowlist. The API deduplicates the list; []/null clears it and omission preserves it. This does not change structured course start dates. Do not insert the current year as a default.

## Validity dates

Enter startTime/endTime as YYYY-MM-DD in Australia/Melbourne. Convert returned ISO timestamps to Melbourne dates before using them; do not take the first ten UTC characters. The API handles daylight saving and UTC storage.

- startTime: omission preserves it. null behaves like clearing the webpage input: it becomes midnight at the start of the current Melbourne day, **not an unlimited or absent start date**.
- endTime: required on the webpage and cannot be cleared through MCP. Stored as the end of the specified Melbourne day; cannot precede startTime.

Check dates, course, location, visa requirement, citizenship, and admission period together when determining applicability, not just the amount. Example: “Add one citizenship to this discount” means read the whole list and merge the new code, not replace the existing list with that code alone. See [saving rules](editing.md).
