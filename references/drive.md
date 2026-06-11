# Google Drive: find collateral and attach files

Use the Google Drive connector to (a) find documents relevant to the reply, and (b) attach or link a file on the draft. Drive holds the user's own working material — PoV reports, test plans, decks, datasheets, SOWs — which is often exactly what a reply needs to reference or send.

## Searching Drive

Use the Drive connector's `search_files` with a structured query. Syntax: `term operator value`, combined with `and` / `or` / `not`. String values are single-quoted.

- Content match: `fullText contains 'megaport'`
- Title match: `title contains 'PoV Report'`
- By type: `mimeType = 'application/vnd.google-apps.presentation'` (Slides), `'application/pdf'`, `'application/vnd.google-apps.document'` (Docs), `'application/vnd.google-apps.spreadsheet'` (Sheets)
- Recent + topic: `fullText contains 'Equity Residential' and modifiedTime > '2026-05-01T00:00:00Z'`

Results include `id`, `title`, `mimeType`, `viewUrl` (shareable link), `owner`, `modifiedTime`, and a `contentSnippet`. To quote from a doc, fetch its text with `read_file_content(fileId)` (returns natural-language text for Docs/Slides/Sheets/PDF). Open a file before quoting it — don't rely on the snippet alone.

## When to bring in Drive

- The message asks for a document the user has ("can you send the PoV report", "share the test plan", "do you have a deck on X").
- A relevant piece of the user's own collateral would strengthen the reply (e.g. attach the latest PoV report when answering a status question).
- The reply references a customer/account that has Drive material — pull it for accurate, current details.

Drive is a supplement. **Rovo and the Netskope docs remain authoritative for product behavior and internal facts** (see grounding.md). Treat the user's Drive docs as accurate for that user's own work (their PoV findings, their notes), not as official product documentation.

## Attaching to an email draft (Gmail)

The Gmail `create_draft` tool accepts `attachments: [{ content, filename, mimeType }]`, where `content` is base64 and the combined size must stay under 25MB.

1. Get bytes: `download_file_content(fileId)` returns base64. For Google-native files (Docs/Slides/Sheets) you must pass `exportMimeType` — e.g. `application/pdf` to attach a PDF copy, or the Office equivalent.
2. Pass it as an attachment with a sensible `filename` and matching `mimeType`.
3. **Fallbacks** — link instead of attach when:
   - The file is larger than 25MB.
   - Attaching fails (some Gmail connector versions don't yet support attachments).
   - It's a living Google-native doc the recipient should see in its current state.
   In those cases, put the file's `viewUrl` in the email body (e.g. "Report here: <url>"). For Google-native files, a link is usually the better choice anyway.

## "Attaching" to a Slack draft

Slack drafts can't carry file uploads through this tool. Include the Drive `viewUrl` as a link in the draft `message` instead.

## Sharing guardrail (important for external replies)

A reply is **external** when any recipient's email domain differs from the user's (or, in Slack, the channel is shared/Slack Connect). Before linking or attaching a file to anyone outside the company, check it's appropriate to share:

- Use `get_file_permissions(fileId)` to see current access. If the file isn't already shared with the recipient and the reply is external, **do not silently widen access** — note it in the chat summary and let the user decide (or attach a PDF copy instead of a link).
- Never attach or link internal-only material (internal notes, other customers' data) to an external recipient. When in doubt, flag it and ask.
