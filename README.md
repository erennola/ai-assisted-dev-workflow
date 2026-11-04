# AI-assisted development workflow

How I work with AI coding agents (Claude Code, Cursor) on production systems: a written workflow, an example `CLAUDE.md`, MCP integrations (Jira, Linear, Confluence, Git), parallel agents, and the knowledge-base structure that keeps long-running work recoverable across sessions.

I'm **[Ezequiel Rénnola](https://www.linkedin.com/in/ezequiel-rennola/)**, a Senior Full Stack Engineer (PHP · React · AWS) based in Buenos Aires. Since 2023 I've used this process day to day on a multi-tenant SaaS platform: production support, data fixes, a legacy-to-Laravel rebuild and security work. This repo is the generic, client-free version of it. Nothing here comes from an employer codebase.

**What it changed in practice:** recurring production bugs on a legacy platform became rare once each fix came with a root-cause write-up and a rule the agent reads every session. On joining a second product, I cleared dozens of abandoned support tickets in my first week.

## The short version

The agent is a fast, tireless pair. I stay the engineer of record.

- **I own decisions, the agent does the legwork.** It reads tickets, explores code, drafts queries, writes code and documents. I approve plans, run anything against production, push, deploy, and decide what a ticket's status is.
- **Plan, then change.** Non-trivial work starts with a written plan and alternatives. No code change without agreement.
- **Production is read-only for the agent.** It writes the SQL; I review and run it, then paste back the results. Local Docker databases are fair game.
- **Verify against data, not beliefs.** "It's the same bug again" is a hypothesis. Before-and-after queries prove a fix.
- **Parallelise the independent, centralise the decisions.** Sub-agents take narrow, independent tasks. One main session keeps the plan and the big picture.
- **Write it down.** Every ticket ends with an incident write-up and an index entry, so the next session (human or agent) starts from knowledge, not from chat history.

## What's in here

| Path | What it is |
|---|---|
| [`WORKFLOW.md`](WORKFLOW.md) | The full workflow: intake, database access rules, planning, development, review, testing, ticket hygiene, close-out, parallel agents. |
| [`case-studies/weekday-only-sync-failure.md`](case-studies/weekday-only-sync-failure.md) | A real incident, anonymized: a sync that only failed during office hours, and how the method found two stacked root causes. |
| [`examples/CLAUDE.md`](examples/CLAUDE.md) | An annotated project memory file for a fictional PHP + React app, showing what is worth putting there and what isn't. |
| [`docs/mcp-jira-linear.md`](docs/mcp-jira-linear.md) | Connecting the agent to Jira and Linear through MCP, and the guardrails that make write access safe. |
| [`docs/knowledge-base.md`](docs/knowledge-base.md) | How to structure agent-readable docs so they still work at 100+ files: folders, headers, size limits, indexes. |
| [`templates/`](templates/) | Copy-paste templates: incident write-up, session handoff, folder index. |

## A ticket, start to finish

An illustrative example. The app, ticket and numbers are invented.

1. **Intake.** I hand the agent `SUP-123: "Certificates not issued after course retake"`. It pulls the ticket through the Jira MCP and asks two clarifying questions.
2. **Diagnosis.** The agent reads the award code path and checks the schema in the repo (it never guesses column names), then gives me one batch of read-only queries. I run them on production and paste back the results.
3. **Finding.** 14 users have a stale completion row from the previous period. The agent proposes two fixes: a data repair alone, or a data repair plus a guard in the reset job. It lists the trade-offs, and I pick the second.
4. **Change.** The agent creates a branch, writes the fix and a test, and reproduces the issue and the fix against a local copy of the data. An automated review pass runs before I open the PR.
5. **Release.** I push, a teammate reviews, I deploy. Verification queries confirm the 14 users now hold certificates and no other rows changed.
6. **Communication.** The agent drafts a Jira comment: finding, fix and PR link, in neutral wording, tagging only the person I named. I post it. The status change is left to the reporter.
7. **Close-out.** The agent writes `incidents/sup-123-certificates-after-retake.md` and adds one line to the index. If the bug revealed a reusable trap, it adds one bullet to `CLAUDE.md`.

## Why bother writing all this down

Agents are only as good as the context they start with. Long investigations outlive any single chat. A workflow file plus a well-structured knowledge base means:

- a new session gets up to speed in minutes by reading two or three short docs, not a 100-message history;
- the same mistake (guessed column names, an unreviewed production write, an over-eager ticket transition) gets fixed once, in the rules, instead of every week;
- teammates can read the same docs the agent reads.

## Contact

[LinkedIn](https://www.linkedin.com/in/ezequiel-rennola/). Open to remote senior full stack roles (US, Canada, LATAM).

## License

[MIT](LICENSE). Use anything here in your own workflow.
