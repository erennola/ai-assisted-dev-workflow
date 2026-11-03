# Workflow: working with an AI coding agent

This is the operating agreement between me (the engineer) and the agent. The agent reads it at the start of every session. It is written for a PHP + React + MySQL stack on AWS, but nothing in it depends on that stack.

---

## 1. Roles

| | Engineer | Agent |
|---|---|---|
| Understands the problem | Brings the ticket, context and constraints | Reads the ticket, code and docs. Asks clarifying questions |
| Production data | Runs every query | Writes queries, never runs them |
| Local data | Approves the approach | Runs queries and changes, announcing each one first |
| Code | Approves the plan, reviews the diff | Plans, implements, tests, commits on a branch |
| Remote and release | Pushes, merges, deploys | Never pushes or deploys |
| Tickets | Owns status, assignee, priority, scope | Reads freely. Posts informational comments when asked |

---

## 2. Process

### 2.1 Intake
- I present the task with whatever I have: ticket key, screenshots, logs, a teammate's message.
- The agent reads the ticket directly through MCP (see [`docs/mcp-jira-linear.md`](docs/mcp-jira-linear.md)), then the relevant docs from the knowledge base.
- The agent asks questions until the problem is clear. Separate **what the reporter literally asked** from **what people believe is happening**. Beliefs often reach us verbally and are often wrong.

### 2.2 Database access
**Production and staging:**
- The agent writes SQL. I review it, run it, and paste the results back.
- **Read before write.** Diagnose with `SELECT`s. Any write is a reviewed script that shows the rows it will touch before touching them, runs inside a transaction, and ends with a verification query.
- **Batch queries.** Give me every query needed for the next step in one message, labelled, so I run them once.
- **Never guess the schema.** Check table and column names in the repo (migrations, install SQL, models) before writing a query. A guessed column costs a full round-trip.

**Local (Docker):**
- The agent may run queries and changes directly, saying what it is about to run before it runs it.
- Prefer a local copy of real (sanitised) data to reproduce production issues.

### 2.3 Planning
- Anything beyond a trivial fix starts with a plan: what changes, where, why, and how we'll know it worked.
- Present **alternatives with trade-offs**, not a single answer.
- **No code changes without agreement.** That applies twice over in shared code that every customer or tenant runs.

### 2.4 Development
- Work on a branch named `<TICKET>-<short-description>`.
- The agent commits. Commit messages are in English and explain the *why*.
- The agent does **not** push. I push.
- PR descriptions cover the summary, root cause, the change, how it was tested, and the risk and blast radius.

### 2.5 Review
- For non-trivial changes, run an **automated review pass before human review**. Fold real findings into a `review:` commit and discard nits. Human reviewers should spend their time on judgment, not lint.
- The agent triages review feedback: what's worth fixing vs. what's over-engineering, with a reason for each.
- Fixes land as new commits on the same branch.

### 2.6 Testing and verification
- Reproduce locally first, then fix, then show the fix with the same reproduction.
- Run the **same verification queries before and after** a data fix, and report counts.
- Staging before production when the change allows it.
- A recurring symptom is not proof of one recurring bug. Confirm the *current* cause against current data.

---

## 3. Language

- **English:** code, identifiers, comments, commits, PRs, branch names, logs, and every knowledge-base document.
- **Conversation** can happen in whatever language the engineer prefers. Mine is Spanish.

---

## 4. Ticket and comment hygiene

The agent can write to the tracker. That's useful and risky, so:

- **Never change lifecycle without explicit confirmation.** Status transitions, assignee, priority, sprint, fix version, and new linked tickets all need my explicit OK in chat first.
- **Neutral wording on lifecycle.** No "closing the loop" or "feel free to close" unless closing was actually decided. Prefer "Fix is in production, back to you for validation."
- **Tag only who I asked to tag.** Pulling more people into a thread is an escalation, and that's my call.
- **No filler.** The thread's readers know the system. Lead with the finding, the question or the next step.
- **No pre-emptive defence.** Don't write "This is not a regression from my change" unless someone asked.
- **Match the reader.** When a teammate will relay the answer to a customer, give them the key points in plain prose rather than a finished script, without internal table names.
- **No code-change proposals in customer-facing comments** without discussing them in chat first. Code changes are scoped and billed work, and a change to shared code affects every customer.
- **Read it back in my voice.** If it doesn't sound like an engineer talking to a peer, rewrite it.

When in doubt, leave it out. Removing text before posting is cheaper than redacting after.

---

## 5. Anti-patterns

- Running anything against production.
- Writing SQL against a schema you haven't looked at.
- Changing code before the approach is agreed.
- Pushing, merging, deploying.
- Transitioning or re-assigning tickets.
- Treating "the client says it's the same bug" as a diagnosis.
- Loading an entire knowledge base or chat archive into context "just in case".

---

## 6. Change checklist

- [ ] Approach agreed before implementation
- [ ] Code, comments and commits in English
- [ ] Reproduced and fixed locally, with the same reproduction
- [ ] Before/after verification queries run, counts reported
- [ ] Automated review pass done (non-trivial changes)
- [ ] PR description: root cause, change, testing, risk
- [ ] Docs updated (incident write-up, index, and `CLAUDE.md` if there's a reusable lesson)

---

## 7. Closing out a ticket

Mandatory when a fix is merged or a diagnosis is delivered:

1. **Incident write-up** in `incidents/<product>/<ticket>-<client>-<topic>.md` ([template](templates/incident.md)). Cover the symptom, root cause, fix with PR links, why it's safe for other tenants, verification against real data, reusable queries and lessons. It's written for future-me, so it can be detailed. Keep it under 20 KB.
2. **Index line** in that folder's `INDEX.md`: link and one-sentence summary.
3. **Memory**, but only when the ticket exposed a reusable trap. Add one bullet to `CLAUDE.md` under *Known traps*, linking the write-up. Most tickets don't need this.

The write-up holds the story, the index makes it findable, and the memory surfaces the one thing worth remembering automatically. Structure details: [`docs/knowledge-base.md`](docs/knowledge-base.md).

---

## 8. Long-running work

For investigations or programs that span many sessions:

- Keep one **status doc** that a new session reads first. It holds the mission, what's done, what's open and the next step. Don't reconstruct state from chat history.
- End each session with a **handoff** ([template](templates/session-handoff.md)): the exact state, the next action, and a ready-to-paste prompt for the next session.
- Session logs go to `archive/`. Anything durable gets distilled into reference, playbook or incident docs.
