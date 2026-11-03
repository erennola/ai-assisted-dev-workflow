# A knowledge base that agents (and people) can actually use

After a few months of AI-assisted work you have dozens of markdown files: investigations, session logs, architecture notes. Kept flat and unstructured, they stop helping. The index grows to 90 KB, new sessions load too much or too little, and stable knowledge gets buried in diaries. This is the structure I settled on after reorganising roughly 100 documents.

## Layout

```
kb/
├── README.md            ← short entry point (~3 KB): folder map, most-used docs, conventions
├── reference/           ← how things work: architecture, schemas, known pitfalls
│   ├── INDEX.md
│   └── quick-reference/ ← lookups: key tables, file paths, code paths
├── playbooks/           ← step-by-step recipes (incl. WORKFLOW.md)
│   └── INDEX.md
├── incidents/           ← one write-up per ticket, grouped by product
│   ├── INDEX.md         ← grouped by client, newest first
│   └── <product>/
└── archive/             ← dated session logs and superseded docs
    └── INDEX.md
```

The split that matters most is **knowledge vs. diary**. A session log ("tried X, then Y, the container died, re-ran Z") is valuable for reconstructing *why*, but it's noise for answering *how*. Logs go to `archive/`, and the durable parts get distilled into the other three folders.

## Rules

**1. Every doc has a header.**

```yaml
---
title: "Certificates not issued after course retake"
summary: "Completion rows survived the award reset; data repair for 14 users plus a guard in the reset job."
updated: 2026-05-12
type: incident            # reference | playbook | incident | session-log
client: "Example Corp"
tickets: [SUP-123]
---
```

The `summary` line is what lets an agent (or `grep`) decide whether to open the file at all.

**2. 20 KB max per file.** Past that, a doc becomes a folder holding a `README.md` (summary plus ordered table of contents) and numbered parts. Small files mean the agent loads only the part it needs.

**3. Two-level index.** The root README lists folders. It only changes when a folder changes. Each folder's `INDEX.md` has one line per doc. Adding a doc means adding one line.

**4. Predictable names.** Lowercase kebab-case. Incidents: `<ticket>-<client>-<topic>.md`, or `adhoc-<yyyy-mm>-<client>-<topic>.md` when there's no ticket. Session logs: `<yyyy-mm-dd>-<topic>.md`.

**5. No sensitive data.** No customer emails or names (use IDs or initials), credentials, hostnames or IPs, and no CSV exports or dumps. The KB gets read into third-party model context and sometimes shared with teammates. Treat it as semi-public.

**6. English.** It's the language of the code, the team and the tools.

## How a session uses it

1. The agent reads `CLAUDE.md` (auto-loaded) and `playbooks/WORKFLOW.md`.
2. Based on the task, it opens one folder `INDEX.md` and picks two or three docs by their summaries.
3. For a split doc, it reads the `README.md` and then only the relevant part.
4. At close-out it writes the incident doc and adds the index line ([WORKFLOW §7](../WORKFLOW.md#7-closing-out-a-ticket)).

Templates: [incident](../templates/incident.md), [session handoff](../templates/session-handoff.md), [folder index](../templates/INDEX.md).
