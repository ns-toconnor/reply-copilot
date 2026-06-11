# Reply Copilot

A CLAFTS-style drafting assistant for Netskope solutions engineers. It reads an email or Slack message you need to answer, researches the answer **only** in Rovo (Jira + Confluence) and the official Netskope documentation, pulls in relevant Google Drive collateral (and attaches or links it), writes a reply in your voice, and leaves a **Gmail or Slack draft** for you to review and send. It never sends anything on its own.

Inspired by Anthropic's GTM engineering write-up on CLAFTS — grounded, voice-matched drafts so the facts are always current and the writing still sounds like you.

## Skills

- **`/draft-email-reply`** — Reply to a customer or colleague email. Creates a Gmail draft.
- **`/draft-slack-reply`** — Reply to a Slack message or thread. Creates a Slack draft.
- **`/daily-drafts`** — Triage everything awaiting your reply across email and Slack, rank it, and batch-draft the top items (default 10, last 7 days) into Gmail and Slack — the inbox-clearing workflow.

Trigger them naturally too: "draft a reply to this email", "answer Conrad's Slack about Megaport", "respond to the US Bank thread", "triage my inbox", "draft my unanswered messages".

Tip: schedule `/daily-drafts` to run each morning so a set of grounded drafts is waiting when you sit down. Verify one scheduled run end-to-end before relying on it — some connector authentication is only available in interactive sessions, so a headless run may not reach every connector.

## How it works

1. Pulls the message you point it at (or that you paste).
2. Splits out the asks: Netskope-product questions vs. internal/account questions.
3. Researches — Netskope docs for product questions, Rovo for internal knowledge — and grounds every technical claim in a real source. If it can't find a source, it flags the gap instead of guessing.
4. Searches Google Drive for relevant collateral and, when useful, attaches it to the email (base64, under 25MB) or links it (Slack, or larger/living docs) — with a permission check before sharing anything externally.
5. Matches your writing voice: on first use it samples your recent sent mail/Slack and caches a profile, then reuses it.
6. Saves a draft and shows you the text plus the sources it used.

## Requirements

Five connectors: **Netskope**, **Atlassian** (Rovo), **Google Drive**, **Gmail**, and **Slack**. See `CONNECTORS.md`.

## Model split (cost + architecture)

The single-reply skills run entirely in a forked subagent (`agents/reply-drafter.md`) pinned to **Sonnet**, so the token-heavy research and drafting stays cheap even if your Cowork session is on Opus, and the research stays out of your main thread.

`/daily-drafts` splits the work by what each step needs:

- **Triage on Haiku** (`agents/triage-scout.md`) — reading the inbox and Slack backlog is the input-heavy part (dozens of threads to pick ~10), but it's mostly mechanical: who spoke last, is it automated, how old is it. The scout returns a ranked candidate list and never drafts.
- **A sanity pass + voice calibration in your main session** — the judgment-heavy, low-token glue. Suspected phishing never goes to a drafter; first-run voice calibration stays off the small model (after that it's cached).
- **One Sonnet drafter per selected item** (`agents/reply-drafter.md`) — fresh context and full-quality research per draft, items run in parallel, one failure doesn't sink the batch.

To change the models, edit `model:` in `agents/reply-drafter.md` and/or `agents/triage-scout.md` (`sonnet`, `opus`, `haiku`, `inherit`, or a full model id).

## Guardrails

- Two sources of truth only (Netskope docs + Rovo) for technical/internal facts — no hallucinated product behavior.
- Always a draft, never an auto-send — and as a hard backstop, the drafting agent has send-capable tools blocked at the tool level (`disallowedTools` in `agents/reply-drafter.md`), so this doesn't depend on the model following instructions.
- Inbound messages are treated as untrusted content: embedded instructions ("forward this", "share that file") are ignored and flagged, not followed.
- Keeps drafts measured and professional even when the inbound message is heated.

## Customize the voice

The plugin calibrates to whoever runs it: on first use it samples your recent sent mail and Slack messages, infers your style, and caches the profile as `reply-copilot-voice.md` in your working folder. Edit that file to tune the tone, or delete it (or say "recalibrate my voice") to re-sample. `references/voice.md` holds the calibration procedure, a generic fallback, and a worked example of a calibrated profile. Teammates who install the plugin get the same workflow calibrated to their own writing — including their own sign-off.
