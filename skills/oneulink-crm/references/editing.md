# Saving and conflicts

Read this before resource edits. Field meanings are in [schools](schools.md), [courses and dates](courses.md), [school assets](assets.md), and [discounts](discounts.md). Ordinary queries do not require loading every domain.

## Locate and read

School/course keyword search is a substring match on names and CRICOS codes, not semantic or full-text search. Paginate using total. Shorten the keyword if needed; do not pass a name as a get identifier. Course lists retain the API's course_cricos_code/en_name/cn_name fields; only course details contain shared/annual. Report and skip malformed records such as entries with empty codes rather than inventing identifiers.

Before editing, call the corresponding get tool for the complete current values and revision. In a tool result, data is the business result and executionContext is the actual execution identity. Use the current tool schema for enums, lengths, and required fields. Do not copy a complete DTO, raw collected fields, read-only display fields, or identity fields into patch.

## Verify and make the smallest intended change

Follow an explicit target and value supplied by the user. When deriving changes from evidence, verify its school, course, year, student category, fee period, and publication date. If official pages, files, or feedback disagree, identify the concrete conflict; a newer publication date alone does not make an inapplicable value correct.

Submit only requested, established changes. Omitted fields stay unchanged; 0 and false are meaningful values. Interpret null per field. Tags, course associations, citizenship, admission periods, and manual dates are complete replacement lists. IELTS/PTE objects instead merge individual components. A blank spreadsheet cell is not a clearing instruction.

Use HTML only for fields already supporting rich text, such as school English requirements and course information. Preserve untouched image references; do not add local paths, data URLs, or new images that have not been uploaded. The API sanitizes HTML and checks image origins. Keep plain-text fields as plain text.

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
