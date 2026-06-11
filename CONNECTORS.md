# Connectors required

This plugin drives your connected tools — it does not bundle any servers. Each person who installs it needs these five connectors enabled in Cowork. The skills refer to tools by name (e.g. `search_netskope_documentation`); Claude maps them to whichever instance you've connected.

| Purpose | Connector | Key tools used |
| --- | --- | --- |
| Product answers | **Netskope** | `search_netskope_documentation` |
| Internal knowledge (Rovo) | **Atlassian** (Jira + Confluence) | `search`, `searchConfluenceUsingCql`, `searchJiraIssuesUsingJql`, `getConfluencePage`, `getJiraIssue`, `fetch` |
| Documents + attachments | **Google Drive** | `search_files`, `read_file_content`, `download_file_content`, `get_file_permissions` |
| Email read + draft | **Gmail** | `search_threads`, `get_thread`, `create_draft`, `list_drafts` |
| Slack read + draft | **Slack** | search, `slack_search_users`, `slack_read_thread`, `slack_read_channel`, `slack_send_message_draft` |

Notes:
- "Rovo" here = your Atlassian connector's search over Jira and Confluence. The plugin only treats **Netskope docs** and **Rovo** as sources of truth for technical and internal facts. Google Drive is used for your own collateral (PoV reports, decks, datasheets) and for attaching/linking files — not as product documentation.
- The plugin **never sends** email or Slack messages — it only creates drafts for you to review and send. As a hard backstop, both bundled agents (`agents/reply-drafter.md`, `agents/triage-scout.md`) block send-capable tools via `disallowedTools` — the triage scout can't even create drafts — so "drafts only" doesn't depend on instructions alone. Tool names vary slightly by connector instance — if your Slack or Gmail connector exposes send/schedule tools under different names, add them to that list.
- **Slack DM coverage:** depending on the connector, Slack search may only reach channels, not DMs. `/daily-drafts` will state in its report when DMs couldn't be triaged rather than implying full coverage.
- If you're on Microsoft 365 or another chat tool instead of Gmail/Slack, the skills will need minor edits to point at those connectors' draft tools.
