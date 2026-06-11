---
name: draft-slack-reply
description: Draft a grounded reply to a Slack message or thread in the user's own voice. Use when the user says "reply to this Slack", "draft a Slack response", "answer someone in Slack", "respond in a channel", "draft slack reply", or points to a Slack message/thread they need to answer. Researches every technical answer in Rovo (Jira + Confluence) and the official Netskope documentation, links relevant Google Drive files, matches the user's Slack style, and creates a Slack draft for review without sending it.
context: fork
agent: reply-drafter
---

# Draft Slack Reply

Produce a Slack reply that is (1) grounded only in Rovo + Netskope docs for any technical/internal claim, (2) written in the user's Slack voice (shorter and more casual than email), and (3) saved as a Slack draft for the user to review and send. Never send automatically.

Before drafting, read:
- `${CLAUDE_PLUGIN_ROOT}/references/drafting.md` — the shared per-item procedure: asks → research → collateral → voice → write → draft.
- `${CLAUDE_PLUGIN_ROOT}/references/grounding.md` — research rules and the two authoritative sources.
- `${CLAUDE_PLUGIN_ROOT}/references/voice.md` — how to match the user's writing style.
- `${CLAUDE_PLUGIN_ROOT}/references/drive.md` — how to search Google Drive and link files.

## Pre-flight check

Before doing anything else, verify the Slack connector is available:
- Check whether `slack_send_message_draft` and `slack_read_thread` (or equivalently named Slack tools) exist in your available tools.
- If they do NOT exist, immediately stop and respond:
  "⚠️ Slack connector is not available in this context. Run `/draft-slack-reply` from the Cowork chat window where your Slack connector is authenticated. Do not run this as a local Claude Code command."
- Do not proceed to Step 1 if the check fails.

## Steps

1. **Get the message and its context.**
   - If the user gives a channel + message link/timestamp, read the surrounding context with the Slack connector: `slack_read_thread` for a thread, or `slack_read_channel` for nearby messages. Use search (`from:@... in:#...`) to locate it if needed.
   - If the user pasted the message, use that. If it's unclear which message to answer, ask once.

2. **Draft it — follow `drafting.md` end to end (Slack path).** Extract and tag the asks (and make the internal/external call — shared/Slack Connect channels are external), research in Netskope docs + Rovo, find a Drive file to link if relevant (Slack drafts can't carry uploads, so link the `viewUrl`), calibrate voice (optionally sample the user's own recent Slack messages — resolve their user id via `slack_search_users` and search `from:<@USER_ID>`), write short and direct in mrkdwn, then create the draft with `slack_send_message_draft` (`channel_id`, `message`, `thread_ts` if in-thread). Do **not** send.

3. **Report back in chat.** Show the drafted text, a one-line note on tone, a short **Sources** list (titles + URLs), any file you linked, and any gaps you couldn't ground. Tell the user the draft is staged in Slack for review.

## Rules

- Two sources only for technical/internal facts: Netskope docs and Rovo. No guessing about product behavior.
- Never auto-send; always leave a draft.
- Keep it measured and professional even if the thread is heated.
- Match the user's natural Slack brevity — don't turn a one-line answer into a paragraph.
