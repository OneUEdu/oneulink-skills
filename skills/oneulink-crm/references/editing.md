# Saving and conflicts

Read this before resource edits. Field meanings are in [schools](schools.md), [courses and dates](courses.md), [school assets](assets.md), and [discounts](discounts.md). Ordinary queries do not require loading every domain.

## Locate and read

School/course keyword search is a substring match on names and CRICOS codes, not semantic or full-text search. Paginate using total. Shorten the keyword if needed; do not pass a name as a get identifier. Course lists retain the API's course_cricos_code/en_name/cn_name fields; only course details contain shared/annual. Report and skip malformed records such as entries with empty codes rather than inventing identifiers.

Before editing, call the corresponding get tool for the complete current values and revision. In a tool result, data is the business result and executionContext is the actual execution identity. Use the current tool schema for enums, lengths, and required fields. Do not copy a complete DTO, raw collected fields, read-only display fields, or identity fields into patch.

## Verify and make the smallest intended change

Follow an explicit target and value supplied by the user. When deriving changes from evidence, verify its school, course, year, student category, fee period, and publication date. If official pages, files, or feedback disagree, identify the concrete conflict; a newer publication date alone does not make an inapplicable value correct.

Submit only requested, established changes. Omitted fields stay unchanged; 0 and false are meaningful values. Interpret null per field. Tags, course associations, citizenship, admission periods, and manual dates are complete replacement lists. IELTS/PTE objects instead merge individual components. A blank spreadsheet cell is not a clearing instruction.

Use HTML only for fields already supporting rich text, such as school English requirements and course information. Preserve untouched image references; do not add local paths, data URLs, or new images that have not been uploaded. The API sanitizes HTML and checks image origins. Keep plain-text fields as plain text.

## Choose a destination before writing

Match each verified fact to its meaning, scope, year and audience. A string or HTML field accepting the value does not make it the right destination.

| Situation | Action |
| --- | --- |
| An accurate field and permitted tool exist | Make the smallest authorized change in that field. |
| The right field exists but its tool or permission is unavailable | Report the missing capability; leave that change unfinished. Do not use another writable field or bypass the MCP. |
| Verified supplementary context affects internal advice or applications | Use consultationNote only if it independently fits that purpose; follow the note-writing section below. |
| No suitable field exists, or the destination/meaning is unclear | Preserve the report in feedback and explain the gap in handlingNote when feedback handling is authorized; otherwise report it to the user. Ask only for the missing decision needed to proceed. |

School descriptions and courseInformation describe the school or course, including its overview, teaching content or learning outcomes. Do not append raw feedback, address inventories, fee tables or other unmapped facts merely because no other tool accepts them. A user's explicit request for relevant descriptive copy permits that editorial change; it does not establish a structured campus relationship or authorize unrelated edits. Preserve untouched description content and image references.

Consultation notes are not a general overflow field either. Saving a note or handling explanation does not complete a missing public-data update. Keep partially handled feedback pending/in_progress, with completed and outstanding items distinguished; do not resolve or close it just because the available tools cannot finish it.

For example, a request to record one theory location and two simultaneously used workshops requires representing distinct locations and their roles for that course. Correct a matching existing campus only when the requested fact really corrects that record. Do not concatenate locations into its address, rename a campus to simulate course roles, or put the inventory in the course description. The current MCP cannot create locations or maintain those course-location roles; report that gap without inventing a field. See [campus capabilities](assets.md#campuses-campuses).

## Write self-contained consultation notes

Apply this when adding or revising OneULink-internal `consultationNote`. An internal reader of the CRM school/course detail may not have the originating conversation or feedback. Write enough factual context to use the note correctly; labels such as “verified school advice”, “special circumstances”, or “confirm first” are not useful notes by themselves.

Include what the evidence establishes:

- The exact school/course and any narrower campus, intake or student group. Course notes are shared across years, so state the applicable year/term when the fact is time-limited.
- The concrete situation and its effect on advice or applications. Distinguish an actual class not running from a cancelled registration or closed admissions; do not invent a cause or generalize a course issue to the whole school.
- Known effective dates, conditions and exceptions. Prefer explicit dates/terms to “currently” or “this year” when the source establishes them. Do not infer validity or verification dates from the edit timestamp.
- Relevant source/confirmation context and date when known, plus what remains to be checked before acting. Keep the note focused on useful internal facts; retain detailed evidence in the edit reason or feedback handling record and avoid unrelated personal details or credentials.

Use a concise paragraph or short factual bullets, without imposing a fixed length or filling empty template sections. Preserve the source's meaning and language. Missing nonessential details may remain unstated; if scope, timing or meaning is too uncertain for a reliable note, ask for that missing fact and keep the unresolved claim in feedback/handling records instead of publishing a guess. “Verified” is not a substitute for the fact and is not a system-generated approval status.

Illustration only, assuming the source confirms each fact: “学校 A 的课程 B 在 2027 年第一学期未实际开班；该课程 CRICOS 注册仍保留。此说明仅涉及该课程该学期，不能据此认定课程已注销或学校其他课程停开。推荐或安排申请前需向院校确认下一期开班情况。” Do not copy these hypothetical facts or dates into real records.

When supplementing an existing note, retain still-valid context, remove superseded statements, and read the merged result as a standalone note before saving. Follow [feedback classification](feedback.md#choose-the-destination-after-verification) when a report is the source.

## Save and retry

Use a separate, stable idempotencyKey, such as a UUID, for each resource edit and provide its reason. Retries of the same request reuse its key, payload, and revision. For courses, send only the revision of each modified group; other resources use the version returned by their own details. Resource revisions are server-issued strings, not timestamps or locally incremented numbers.

- revision_conflict: reread and compare concurrent changes with the intended edit. Do not simply substitute the latest version and overwrite. Non-conflicting changes still within authorization may form a new operation.
- idempotency_conflict: the same key was used for a different payload. Check the previous operation; this is not an ordinary version conflict.
- Timeout/disconnection: the outcome is unknown. Read back first; if retrying, reuse the same key and payload. Do not repeatedly generate new keys and resubmit.
- invalid_country_code, invalid_year, or fields errors: correct established mistakes at the reported field paths. Check the source year as well; do not substitute the first allowed year.
- If the outcome remains unclear, failures repeat, or permission is lost, stop that operation and report what is needed to resume.

feedback_update uses an integer version and no idempotency key. Follow the [feedback rules](feedback.md), not resource retry semantics.

## Determine completion

Check the business result and persisted values, reading back with get when needed. HTTP 200, a tool returning without an exception, or an auditId does not prove website synchronization. For syncState=pending/failed, report that CRM saved the change while website sync is pending/failed. Do not resave to trigger synchronization or invent an unavailable sync-retry tool.

Report multi-step outcomes individually, preserving identifiers, years, before/after values, auditId, and reasons for unfinished work. Do not describe partial success as complete batch success.
