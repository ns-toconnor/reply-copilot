# Grounding: Rovo + Netskope docs only

Every factual or technical claim in a draft MUST come from one of exactly two sources. Do not answer technical questions from memory.

## The two allowed sources

1. **Netskope product documentation** — for anything about how Netskope products, features, configuration, limits, versions, or troubleshooting work. Use the Netskope connector's `search_netskope_documentation` tool. The official docs are the source of truth and stay current as the product ships, so prefer them over recollection.

2. **Rovo (Atlassian: Jira + Confluence)** — for internal knowledge: account/customer context, POVs, known issues, feature requests, internal processes, runbooks, decisions, and "what did we say / what's the status" questions. Use the Atlassian connector's tools:
   - `search` — Rovo unified search across Confluence + Jira (start here).
   - `searchConfluenceUsingCql` — targeted Confluence page search (CQL). Useful: `text ~ "<topic>" AND type = page ORDER BY lastmodified DESC`.
   - `searchJiraIssuesUsingJql` — targeted Jira issue search (JQL). Useful for feature requests / known issues: `text ~ "<topic>" ORDER BY updated DESC`, or by account/key.
   - `getConfluencePage` / `getJiraIssue` / `fetch` — pull the full body of a promising hit before quoting it.

## Procedure

1. **Extract the asks.** List every question or request in the message. Separate Netskope-product questions (→ docs) from internal/account/process questions (→ Rovo). A message can have both.
2. **Search before writing.** Run the relevant searches. Open the top hits (`fetch` / `getConfluencePage` / full doc content) so you quote real wording, not the snippet.
3. **Ground each claim.** For every technical statement in the draft, keep the source URL it came from. Prefer the most recently updated source.
4. **Handle gaps honestly.** If neither Rovo nor the Netskope docs answer a question:
   - Do NOT invent an answer or guess at product behavior.
   - In the draft, either omit that point or write a clearly-marked placeholder like `[need to confirm — no source found]` so the user can fill it in.
   - Tell the user in chat which asks you couldn't ground.
5. **Non-technical content is fine without a source** — greetings, scheduling, logistics, acknowledgements, relationship glue. The grounding rule applies to claims about Netskope products and internal facts, not to ordinary conversation.

## A note on Google Drive

Google Drive (see `drive.md`) is a third place to look — but for the user's **own** collateral: PoV reports, test plans, decks, datasheets they've written. Treat those as accurate for that user's work, and use them to find files to attach or link. They are not a substitute for the two authoritative sources above when answering questions about product behavior or internal process — keep using Netskope docs and Rovo for that.

## Citing sources

- In the **chat summary** (not necessarily in the email body), list the sources you used as a short "Sources" list with titles and URLs, so the user can verify before sending.
- In the **draft body**, weave facts in naturally. Only include a doc link in the body if it genuinely helps the recipient (e.g. "full steps here: <url>"). Don't dump citations on a customer.
- Never cite a source you didn't actually open and read.
