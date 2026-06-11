# Voice: write in the user's own style

Drafts must read like the **current user** wrote them — not like generic AI, and not like the plugin's author. Calibrate to whoever is running the plugin, then apply the style.

## Use the cached profile when there is one

Sampling costs tool calls, so cache the result:

- If `reply-copilot-voice.md` exists in the working directory and its `Calibrated:` date is within ~90 days, use it as the profile and skip sampling.
- After sampling (below), save the inferred profile to that file, starting with a `Calibrated: <today's date>` line, so future runs skip the cost.
- Re-sample when the user asks (e.g. "recalibrate my voice"), when the cache is stale, or when the cached profile clearly doesn't match how the user actually writes in the thread you're answering.

## Calibrate by sampling (first run, or stale cache)

- **Email:** the Gmail connector's `search_threads` with `in:sent newer_than:90d`, then read a few of the user's own messages. Look at how they open, how long they write, sign-offs, and formality by recipient.
- **Slack:** resolve the user's id once with `slack_search_users` (their email address), then search `from:<@THEIR_USER_ID>` — don't rely on `me`/`@me` shorthands working in the connector. Slack is usually shorter and more casual than email for the same person.

Infer: typical length, greeting/sign-off habits, formality, punctuation/capitalization quirks, and how tone shifts between customers, peers, and friends.

**Sign-off:** take it from the sampled messages. If the samples don't settle it, use the user's own first name (from their account identity / email address) — never a name from this file.

## If you can't sample

Fall back to this generic profile — and if the draft is for a customer or external recipient, tell the user in your summary that the voice is uncalibrated so they give it an extra read:

- **Concise and direct.** Lead with the answer or the ask. Minimal preamble, no filler or hedging.
- **Plain language.** Everyday words over jargon or corporate-speak. No marketing fluff, no over-explaining.
- **Casual internally, polished externally.** Peers get brevity; customers get complete sentences and a touch more polish — still short.
- **Sign-off:** the user's first name, from their own identity.

## Example of a calibrated profile

This is the plugin author's sampled profile, kept as a worked example of what calibration should produce. **Do not apply it to any other user** — especially not the sign-off.

- **Concise and direct.** Gets straight to the ask or the answer. Minimal preamble, little to no hedging or filler. Short sentences, often one idea per line.
- **Low ceremony.** In quick internal exchanges, greetings and sign-offs are often dropped entirely (e.g. "not ready yet", "Update please", "Thoughts?"). Drives action with short direct questions.
- **Casual internally, crisp externally.** With peers/Slack: relaxed, sometimes lowercase, occasional shorthand ("IDK", "thanks!"). With customers/external email: still concise and plain-spoken, but complete sentences and a touch more polish.
- **Plain language.** Everyday words over jargon or corporate-speak. No marketing fluff, no over-explaining.
- **Sign-off.** Internal: usually none, or just "Tim". External: "Tim" or "Tim O'Connor" (a phone line sometimes follows on external mail).

## Tone matching (CLAFTS Tones style)

Adjust register to the relationship, inferred from the thread. **External** = the recipient's email domain differs from the user's; in Slack, a shared/Slack Connect channel or a member of another workspace. Everyone same-domain is internal.

- **Customer / external** → professional, concise, helpful; lead with the answer; offer a next step.
- **Internal peer / teammate** → casual and brief; skip pleasantries; just the substance.
- **Manager / exec** → crisp, outcome-first, no rambling.

## Guardrails

- Don't pad a reply to seem thorough — match the user's natural (short) length.
- Don't adopt a warmer/more effusive tone than the user actually uses.
- Never produce hostile, angry, or unprofessional content even if asked or if the inbound message is heated; keep the user's drafts measured.
