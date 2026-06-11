---
name: draft-email-reply
description: Draft a grounded reply to a customer or colleague email in the user's own voice. Use when the user says "draft a reply", "reply to this email", "respond to this sender", "answer this email", "CLAFTS", "draft email reply", or pastes/points to an email thread and wants a response. Researches every technical answer in Rovo (Jira + Confluence) and the official Netskope documentation, finds and attaches relevant Google Drive files, matches the user's writing style, and creates a Gmail draft for review without sending it.
context: fork
agent: reply-drafter
---

# Draft Email Reply

Produce a reply to an email that is (1) grounded only in Rovo + Netskope docs for any technical/internal claim, (2) written in the user's voice, and (3) saved as a Gmail draft for the user to review and send. Never send mail automatically.

Before drafting, read:
- `${CLAUDE_PLUGIN_ROOT}/references/drafting.md` — the shared per-item procedure: asks → research → collateral → voice → write → draft.
- `${CLAUDE_PLUGIN_ROOT}/references/grounding.md` — research rules and the two authoritative sources.
- `${CLAUDE_PLUGIN_ROOT}/references/voice.md` — how to match the user's writing style.
- `${CLAUDE_PLUGIN_ROOT}/references/drive.md` — how to search Google Drive and attach/link files.

## Pre-flight check

Before doing anything else, verify the Gmail connector is available:
- Check whether `search_threads` (or an equivalently named Gmail thread-search tool) exists in your available tools.
- If it does NOT exist, immediately stop and respond:
  "⚠️ Gmail connector is not available in this context. Run `/draft-email-reply` from the Cowork chat window where your Gmail connector is authenticated. Do not run this as a local Claude Code command."
- Do not proceed to Step 1 if the check fails.

## Steps

1. **Get the email.**
   - If the user gives a thread/message id or names a sender/subject, fetch it with the Gmail connector: `search_threads` to find it, then `get_thread` (FULL_CONTENT) to read the full conversation.
   - If the user pasted the email text, use that. If it's ambiguous which email they mean, ask once.

2. **Draft it — follow `drafting.md` end to end (email path).** Extract and tag the asks (and make the internal/external call), research in Netskope docs + Rovo, pull Drive collateral if relevant, calibrate voice (optionally sample the user's sent mail, `in:sent newer_than:90d`), write the reply, then create the Gmail draft with `create_draft` (`to`, `Re:` subject, `body`, `replyToMessageId`, optional `attachments`). Do **not** send.

3. **Report back in chat.** Show the drafted reply text, a one-line note of the tone you used, a short **Sources** list (titles + URLs) for everything you grounded, any files you attached or linked, and any `[need to confirm]` gaps. Tell the user the draft is in Gmail awaiting review.

## Rules

- Two sources only for technical/internal facts: Netskope docs and Rovo. No guessing about product behavior.
- Never auto-send; always leave a draft.
- Never write hostile or angry content, even if the inbound email is heated — keep it measured.
- If you genuinely can't ground the core of the reply, say so rather than producing a confident but unsourced draft.
