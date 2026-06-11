# Drafting one reply (shared per-item procedure)

The core procedure shared by `draft-email-reply`, `draft-slack-reply`, and `daily-drafts`. The invoking skill handles finding the message and reporting back; this file covers everything in between. Keeping it in one place means the single-reply and batch skills can't drift apart.

## 1. Extract the asks

List every question or request in the message/thread. Tag each as a Netskope-product question (→ Netskope docs) or an internal/account/process question (→ Rovo). A message can have both.

**Internal vs. external (decide it here — tone and sharing rules depend on it):** a recipient is **external** if their email domain differs from the user's own domain, or, in Slack, if the conversation is in a shared/external (Slack Connect) channel or with a member of another workspace. Same-domain colleagues are internal. This call drives the tone register (voice.md) and the Drive sharing guardrail (drive.md). Also note the relationship (customer, peer, manager) for register.

## 2. Research (grounding.md)

Search before writing.

- Netskope-product questions → `search_netskope_documentation`; open the best hit's full content and quote real wording, not the snippet.
- Internal/account/known-issue/status questions → Rovo: `search`, then `searchConfluenceUsingCql` / `searchJiraIssuesUsingJql`; open top hits with `fetch` / `getConfluencePage` / `getJiraIssue`.
- Keep the source URL behind every technical claim. If a question can't be grounded in either source, do not invent an answer — mark it `[need to confirm — no source found]` and flag it.

## 3. Collateral (drive.md)

If the message asks for a document, or one of the user's own files (PoV report, test plan, deck, datasheet) would strengthen the reply: search Drive with `search_files`, confirm the right file, read it (`read_file_content`) if you'll reference its contents, and note its `id`, `title`, and `viewUrl`. Run the external-sharing permission check before attaching or linking anything to an external recipient. Skip this step if nothing is relevant.

## 4. Voice (voice.md)

Apply the calibrated voice profile — cached, freshly sampled, or passed in by `daily-drafts` (don't re-sample if a profile was handed to you). Match the user's length, formality, greeting, and sign-off for this recipient type. Slack skews shorter and more casual than email for the same person.

## 5. Write

- Lead with the answer. Match the user's natural (short) length — don't pad.
- Ground every technical statement in step 2's sources.
- **Email:** put a doc link in the body only if it genuinely helps the recipient. The reply will be appended above the prior thread automatically.
- **Slack:** an inline "docs:" link is fine (the audience is usually technical). Use mrkdwn. Reply in-thread when the original is part of a thread. Include a Drive file's `viewUrl` as a link — Slack drafts can't carry uploads.

## 6. Create the draft (never send)

**Email** — Gmail `create_draft`:
- `to`: recipient address(es), plain `user@example.com` form only.
- `subject`: "Re:" followed by the original subject.
- `body`: the drafted reply.
- `replyToMessageId`: the **message id** of the email you're replying to (from `get_thread`), so it threads correctly.
- `attachments` (optional): `download_file_content` to base64, pass `{ content, filename, mimeType }`, combined under 25MB. For Google-native files, export to PDF or link the `viewUrl` instead — follow drive.md.

**Slack** — `slack_send_message_draft`:
- `channel_id`: the channel (for a DM, the user id from `slack_search_users`).
- `message`: the drafted text (mrkdwn).
- `thread_ts`: the parent message timestamp if replying inside a thread.
- Only one draft per channel is allowed — on `draft_already_exists`, don't overwrite; tell the user an existing draft is in the way.

## Per-item record

For each draft, keep: where the draft is, the tone used, sources (titles + URLs), files attached or linked, and any `[need to confirm]` gaps. The invoking skill's report step needs these.
