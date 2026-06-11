---
name: daily-drafts
description: Triage everything awaiting your reply and batch-draft responses. Use when the user says "/daily-drafts", "draft my unanswered emails", "triage my inbox", "clear my inbox", "draft replies to everything", "daily drafts", "morning drafts", or "batch draft my messages". Finds unanswered Gmail threads and Slack messages/threads addressed to the user, filters out noise, ranks by priority, then auto-drafts grounded replies (Netskope docs + Rovo, with Drive collateral) into Gmail and Slack for review — never sending.
---

# Daily Drafts (batch triage + draft)

Clear the backlog in one pass: find what's genuinely awaiting the user's reply across email and Slack, rank it, and leave a grounded draft for each of the top items.

**Architecture:** this skill runs in the main session and orchestrates two kinds of subagents (subagents can't spawn subagents, so the fan-out happens from here):

1. **`triage-scout` (Haiku)** — does the input-heavy inbox/Slack reading: gather, filter, rank. Returns a structured candidate list.
2. **`reply-drafter` (Sonnet)** — one per selected item: research, voice, write, create the draft. Fresh context each, run in parallel, one failure can't sink the batch.

You do only the cheap glue here: pre-flight, dispatching, a sanity pass over the triage list, voice calibration, and the final report. Never read the whole inbox, research, or draft inline in this session.

Read `${CLAUDE_PLUGIN_ROOT}/references/voice.md` now — you handle the voice profile in Step 3 and pass it to every drafter. The drafters read the other references themselves.

**Defaults (override if the user specifies):** look back **7 days**, draft the top **10** items total across email + Slack. Honor instructions like "email only", "just the top 5", "last 3 days", "skip Slack".

## Pre-flight check

Before doing anything else, verify the connectors are available:
- Check whether the Gmail tools (`search_threads`) and Slack tools (`slack_send_message_draft`, `slack_read_thread`) — or equivalently named connector tools — exist in your available tools.
- If NEITHER connector is available, immediately stop and respond:
  "⚠️ Gmail and Slack connectors are not available in this context. Run `/daily-drafts` from the Cowork chat window where your connectors are authenticated. Do not run this as a local Claude Code command."
- If only ONE is available (and the user didn't already scope the run to that side, e.g. "email only"), tell the user which connector is missing, triage only the available side, and repeat the limitation in the Step 5 report.
- Do not include a side whose connector failed the check.

## Step 1 — Triage (Haiku subagent)

Spawn one `triage-scout` subagent (this plugin's agent). Its prompt must include: the lookback window, the item cap N, the scope (email + Slack or one side), and the user's email address. It gathers candidates, filters noise and suspected phishing, ranks by priority, and returns the structured list described in its agent file — selected items with identifiers, relationship, priority, and notes, plus the skipped/blocked/overflow lists and a DM-coverage note.

## Step 2 — Sanity-check the list

The triage ran on a small model to keep it cheap, so give its output a quick judgment pass here (you're only reading summaries, not threads):

- Does the ranking match what you know from this conversation and the user's instructions? Promote/demote items if it's clearly off (e.g. a named active customer ranked below a routine internal ping).
- Anything triage flagged as "suspected spam/phishing" or "unsure" stays undrafted — those go in the report for the user to eyeball, not to a drafter.
- If the list looks implausible (zero candidates in a busy week, or obvious duplicates), say so and consider re-running triage with adjusted scope rather than silently proceeding.

## Step 3 — Voice profile (once, in this session)

Follow `voice.md`: if `reply-copilot-voice.md` exists in the working directory and is fresh, use it. Otherwise calibrate **here in the main session** — sample the user's sent mail/Slack per voice.md, infer the profile, and save the cache. (Calibration is a style-judgment task; don't delegate it to the Haiku scout.) Either way, produce a short voice profile (length, formality, greeting/sign-off habits by recipient type) as text to paste into every drafter's prompt. Calibrate exactly once — drafters must not re-sample.

## Step 4 — Draft each item in its own subagent (Sonnet)

For every selected item, spawn one `reply-drafter` subagent; run several in parallel where you can. Each drafter's prompt must include:

- **What to draft**: the identifiers from triage — for email, the Gmail thread id and the message id to reply to, plus sender and subject; for Slack, the channel id, message `ts` (and `thread_ts` if in a thread), plus who's asking and the topic.
- **Triage context**: the relationship (customer / peer / manager), internal vs. external, priority, and the scout's notes (heated tone, deadline, prior promises in the thread).
- **The voice profile** from Step 3.
- **The references to read first**, as full paths: `${CLAUDE_PLUGIN_ROOT}/references/drafting.md`, `${CLAUDE_PLUGIN_ROOT}/references/grounding.md`, `${CLAUDE_PLUGIN_ROOT}/references/drive.md`.
- **Instructions**: follow `drafting.md` end to end; keep research lean (a handful of targeted searches, open only the best hits); ground every technical claim in Netskope docs or Rovo and mark anything unsourced `[need to confirm — no source found]` rather than guessing; create the draft (Gmail `create_draft` with `replyToMessageId`, or `slack_send_message_draft` with `thread_ts`) and **never send**; return the per-item record — draft location, tone used, sources (titles + URLs), files attached/linked, and any gaps.

If a drafter fails, errors, or returns nothing usable, report that item as undrafted and move on — never let one item block the rest, and never retry by drafting it yourself in this session.

## Step 5 — Report

Give the user a compact triage summary, e.g. a table:

| # | Source | From / Channel | Subject / topic | Priority | Status |
|---|--------|----------------|-----------------|----------|--------|

For each drafted item, note any attached/linked Drive file and any `[need to confirm]` gaps. Then list, separately: items skipped as noise/spam (spam/phishing with the scout's reasons, so the user can verify), items skipped because a draft already existed, items blocked by the one-draft-per-channel limit, any drafter failures, and any candidates left undrafted (over the cap). If DMs couldn't be searched, say so. End by telling the user the drafts are waiting in Gmail and Slack for review.

## Rules

- Never send anything — drafts only (both agents also have send-capable tools blocked at the tool level).
- Always delegate: inbox reading to `triage-scout`, drafting to `reply-drafter`. The main session only orchestrates, calibrates voice, and reports.
- Two authoritative sources (Netskope docs + Rovo) for technical/internal facts; no guessing about product behavior.
- Don't reply to spam/phishing or automated mail — suspected cases go to the user, not to a drafter.
- Respect the cap; don't silently draft dozens of messages.
- Never duplicate: a thread/channel that already has a draft is done until the user clears it.
- Keep every draft measured and professional, even for heated threads.
- Be honest about coverage: report what wasn't searched, blocked, or drafted.
