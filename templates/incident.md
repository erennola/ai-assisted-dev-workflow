---
title: "<TICKET> — <client> — <symptom in plain words>"
summary: "<one sentence: cause and what was done>"
updated: YYYY-MM-DD
type: incident
client: "<client>"
tickets: [<TICKET>]
---

# <TICKET> — <client> — <symptom in plain words>

> **Status:** diagnosed | fixed in production | waiting on <who> for <what>
> **PRs:** #<n> · **Reporter:** <role, not a customer's personal name>

## Symptom
What the reporter saw, in their terms. Note separately any belief that isn't in the ticket ("they think the reset is broken"), and where it came from.

## Root cause
The mechanism, with the code path (`path/to/File.php::method`) and the data that proves it.

## Precedents
| Ticket | Same pattern? | Outcome |
|---|---|---|
| <KEY> | <how it matches or differs> | <cause / fix> |

## Fix
- Code: what changed and why this approach (and which alternatives were rejected).
- Data: the repair script, rows touched, transaction and verification.
- Blast radius: why it's safe for other tenants/customers.

## Verification
Before/after queries with counts. Staging and production checks.

## Reusable queries
```sql
-- <what this answers>
SELECT ...;
```

## Lessons
One to three bullets. If one is a reusable trap, add it to `CLAUDE.md` → *Known traps*.
