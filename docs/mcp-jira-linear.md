# Jira and Linear through MCP

[MCP](https://modelcontextprotocol.io) (Model Context Protocol) lets the agent talk to the issue tracker directly: it reads tickets, comments and linked issues, searches, and (if you allow it) writes. That removes the copy-paste step from every intake, and it's also where an agent can do the most visible damage. This doc covers setup and the guardrails I use.

## Setup (Claude Code)

Both vendors host official remote MCP servers with OAuth. Nothing runs locally and there are no API tokens in config files.

```bash
# Linear
claude mcp add --transport http linear https://mcp.linear.app/mcp

# Atlassian (Jira + Confluence)
claude mcp add --transport http atlassian https://mcp.atlassian.com/v1/mcp
```

Then, inside a Claude Code session, run `/mcp` and complete the OAuth flow in the browser. Endpoints change from time to time, so confirm them in the vendors' docs ([Linear](https://linear.app/docs/mcp), Atlassian's Remote MCP Server page) before relying on these.

**Things that commonly go wrong**
- *Atlassian returns 403 "app not installed".* A site admin has to allow the Atlassian MCP app on your Jira site. Until they do, the fallback is the REST API with a personal token, kept in an environment variable and never in the repo.
- *Scope.* OAuth gives the agent **your** permissions. If you can close a ticket, so can the agent. The guardrails below are what stop it.

## What the agent uses it for

| Task | Typical calls | Write? |
|---|---|---|
| Intake: read the ticket, comments, attachments list, linked issues | get issue, get comments | No |
| Find prior art: "has this symptom been reported before?" | JQL search / Linear search by text, component, label | No |
| Cross-check a fix against a backlog of historical tickets | search, then read each | No |
| Post a recap after a fix (finding, PR link, next step) | add comment | **Yes, when asked** |
| File issues found during a test campaign | create issue | **Yes, after review** |

Reading is where most of the value is. In a rebuild project, for example, "check every historical ticket for customer X against the new system" becomes a structured sweep the agent can run and document, instead of a week of tab-switching.

## Guardrails

These live in [`WORKFLOW.md`](../WORKFLOW.md) §4 so the agent reads them every session. In short:

1. **Lifecycle is human-owned.** No status transitions, assignee, priority, sprint or fix-version edits without explicit confirmation in chat.
2. **Comments only when asked**, and I review drafts before they're posted to customer-visible threads.
3. **Tag only the people I name.**
4. **Neutral lifecycle wording.** "Fix is in production, back to you for validation", not "closing this out".
5. **Issue creation in batches I've reviewed.** During a test campaign the agent drafts issues into a doc first: title, steps, expected vs. actual, evidence. I approve the batch, then it files them and writes the keys back into the doc.
6. **Never paste secrets or personal data** (customer emails, tokens, dumps) into tickets. Reference them by ID.

## Patterns that work well

- **Ticket → doc → ticket.** The full investigation lives in the incident write-up in the repo or knowledge base. The ticket gets a short summary and a link. Tickets stay readable, and the detail stays searchable.
- **Linear as the durable record, local docs as working memory.** On a multi-session program, Linear holds the issues while a status doc holds what a new session needs to resume. Neither tries to be the other.
- **Batch reads before a meeting.** "Summarise every open ticket for client X with its last comment and who it's waiting on" is a 30-second ask.
