# School records

Locate with schools_list, read with school_get, and edit with school_update. schoolCode is the school's CRICOS/resource code, id is a database record identifier, and rtoCode belongs to a separate registration scheme. They are not interchangeable.

## Fields

| Field | Meaning and actual behavior |
| --- | --- |
| enName / cnName | English/Chinese name（英文/中文名称）; write each in its named language. The English name cannot be cleared; null clears the Chinese name. |
| enDescription / cnDescription | Public English/Chinese school introduction（英文/中文简介）; write each in its named language. Plain text; null clears it. Not fallback storage for missing fields/tools or raw feedback; preserve unrelated existing content and use [destination selection](editing.md#choose-a-destination-before-writing). |
| url | School website URL（学校官网）. Validated as a URL by the backend; null clears it. |
| rtoCode | RTO registration code, stored as a string. Do not infer it from the name or CRICOS code. |
| englishEntryRequirements | School English entry requirements（英语入学要求）. Supports the existing rich-text HTML format; independent of annual course English notes and scores. |
| applicationFee | School application fee（申请费（AUD））, a non-negative integer in AUD. Distinct from a course registration fee（注册费）. 0 explicitly means zero; null means unset. |
| isReductionApplicationFee | Application fee waiver（减免申请费）: true=waived, false=not waived, null=unset. CRM does not change the recorded fee or conditions. The public school application card, course application steps, and promotional course quotes display 0 when waived; do not overwrite the recorded original fee with 0 merely because of that display. |
| reductionApplicationFeeConditions | Application fee waiver conditions（减免条件）, an independent plain-text field. Setting/clearing it does not toggle the waiver. The public website does not return these conditions when the waiver is off. |
| tags | Complete list of Chinese tags（学校标签）; see below. |

logo and rawApplicationFee may appear in details but are not writable through this MCP. Do not copy the raw application fee or other display values into applicationFee unless explicitly requested.

## Tags replace the complete list

Details may return multilingual objects. Take each item's zh-CN name and description and submit `[{name, description}]`. Preserve all untouched tags when adding or editing one. []/null clears every tag; omitting tags preserves them.

The API retains translations for unchanged Chinese content. Changed Chinese content does not retain obsolete translations. Identical Chinese names with conflicting content are rejected; merge or clarify them rather than reordering to bypass the conflict.

Example: “Add one school tag and leave the rest unchanged” means read the full list, merge the new item, and submit the complete list, not just the new item. See [saving and conflicts](editing.md).

## Consultation notes（学校咨询提示）

`consultationNote` is a plain-text supplement (maximum 5,000 characters) for the school as a whole, used only inside OneULink under the existing school read/edit permissions. Neither OneUEdu MCP nor public website pages return it, even for connected OneUEdu accounts. Do not copy internal notes into public fields or external replies merely because a query user requests school information. Keep it relevant to internal consultation; do not store credentials, unrelated personal information or agent instructions. `consultationNoteUpdatedAt` is a read-only UTC timestamp of the note, not a new verification date.

Read the current note and revision before `school_update`. Omission preserves it; null or whitespace clears it. A submitted note replaces the entire note, so preserve other valid advice when incorporating a new fact. Keep verified conditions and applicable dates. Remove superseded statements rather than accumulating contradictions. Saving a note does not change public descriptions, fees or enrollment status. Read it back through the CRM get tool; it is not published to the website.

Before composing or supplementing a note, follow [self-contained note writing](editing.md#write-self-contained-consultation-notes) so readers understand the actual facts, scope, known dates/conditions and implications without the original feedback. For advice that applies only to a course, use that course's shared.consultationNote instead. For information originating in feedback, classify and verify it using [the feedback workflow](feedback.md), and attach feedbackIds to the original resource edit.
