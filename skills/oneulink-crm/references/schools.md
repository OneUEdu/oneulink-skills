# School records

Locate with schools_list, read with school_get, and edit with school_update. schoolCode is the school's CRICOS/resource code, id is a database record identifier, and rtoCode belongs to a separate registration scheme. They are not interchangeable.

## Fields

| Field | Meaning and actual behavior |
| --- | --- |
| enName / cnName | English/Chinese name（英文/中文名称）. The English name cannot be cleared; null clears the Chinese name. |
| enDescription / cnDescription | English/Chinese description（英文/中文简介）. The webpage uses plain-text inputs; null clears them. |
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
