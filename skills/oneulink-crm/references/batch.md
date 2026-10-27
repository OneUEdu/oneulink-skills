# Batch updates

Map the supplied file to explicit school codes, course codes, years, target fields, and old/new values. Retain source filenames and row numbers; identify duplicates and contradictions. Do not guess missing identifiers, years, currencies, or fee periods.

Check the current tool schema and target details. Read only the relevant domain references and [saving rules](editing.md); exclude unknown fields from patch. Treat blank cells as unspecified. Use null only when the source or user explicitly requests clearing and the field allows it. Distinguish total tuition from weekly rates, onshore international students from domestic students, and annual course records from cross-year dates. Explicit authorization for a defined batch does not require approval for each row. When additional effects or uncertain rows appear, proceed with the established portion within scope and report what needs clarification.

For each row, read its current revision, generate and record a separate idempotency key, then write sequentially. Stop the batch if permission is lost. Retain a conflicting row for review while allowing unrelated, authorized rows to proceed. After a network timeout, retain the original key and read back to establish the outcome. Do not assume batch-wide transactions or all-or-nothing success.

Report per-row results and counts for succeeded, conflicted, failed, and unexecuted rows, with resource identifiers, years, and auditId. Retain resumable row status without tokens or sensitive personal information in result files.

An agent may read and parse a local file, then call existing editing tools row by row within authorization. Reading a file does not upload or save it. Server-side fee extraction, attachment upload, and the original webpage's batch-submit workflow are not exposed through this MCP. Use the original CRM page if those specific workflows are needed; do not invent tools.
