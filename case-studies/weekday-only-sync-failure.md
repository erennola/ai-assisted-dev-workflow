---
title: "Case study: the sync that only failed on weekdays"
summary: "Completions silently stopped syncing during business hours after a database migration. Root cause: report queries holding locks for hours plus a write pattern that MySQL 8 gap locking turned into a contention point."
updated: 2026-11-03
type: case-study
---

# Case study: the sync that only failed on weekdays

> Anonymized from a real production incident on a multi-tenant learning platform (PHP, MySQL 8 on AWS RDS). Table and module names are generic, numbers are rounded, and nothing identifies the client.

## Symptom

Learners finished online courses and the course record correctly showed *complete*. But for some of them the platform's progress tracking never updated, so certificates weren't issued and manager dashboards were wrong. No errors appeared in any log.

Over ten days, about 150 learners and almost 600 course completions were affected.

## The pattern that didn't make sense

The first step was not a fix. It was getting the shape of the failure out of the data. The agent wrote a batch of read-only queries (failure rate by day, by hour, by course, by completion duration). I ran them on production and pasted the results back.

| | Failure rate |
|---|---|
| Weekdays | 12–20% |
| Saturday and Sunday | **0%** |
| 00:00–10:00 and 21:00–23:00 | **0%** |
| 17:00 on the worst day | **86%** |

It wasn't about volume either. The busiest day of the period had zero failures, and an early-morning hour with ~90 completions was clean. Course length made no difference. The same learner could have one completion sync and the next one fail.

So something that only happens during office hours was getting in the way of a write.

## Ruling things out

- **Code changes?** None deployed in the window.
- **Data?** Every affected enrollment was valid and every tracking key existed.
- **What changed?** A schema maintenance script had run a few days before the first failure, part of cleaning up after the MySQL 5.7 → 8.0 upgrade.

A theory that fit all of this: some resource contention that only office-hours activity triggers. But that was still only a theory.

## Reproduction

The proof came by accident. During a business-hours peak, while trying to backfill a single affected learner (four rows), the insert failed:

```
Error 1205: Lock wait timeout exceeded; try restarting transaction
```

That's exactly what the sync handler hit. The handler didn't catch the exception, which is why the failures were silent.

`SHOW PROCESSLIST` at the same moment showed five copies of the same report query, **running for 1 to 6+ hours each**. They came from the manager dashboard's progress report, which joins the progress table against course content for every learner in scope. Managers open that report during their working day. Nobody opens it at night or on weekends.

## Root cause (two layers)

1. **Missing indexes.** The maintenance script had silently dropped two foreign keys on the progress table. The indexes behind those keys were what made the report fast. Without them the report degraded from seconds to hours, holding locks the whole time.
2. **A write pattern that MySQL 8 punishes.** The sync used a self-referential `REPLACE INTO … SELECT … FROM same_table`. Under MySQL 8's gap locking, that statement contends far more than it did on 5.7.

Restoring the foreign keys fixed the report but **not** the sync, and failures continued for two more days. That's why "verify the fix against data" is a rule and not a suggestion. The second layer only showed up because the numbers didn't go to zero.

## Fix

```sql
-- Before: self-referential REPLACE, contention-prone on MySQL 8
REPLACE INTO progress_state (item_id, user_id, state, updated_at)
  (SELECT item_id, user_id, state, updated_at FROM progress_state
   WHERE user_id = :user AND item_id = :item AND state = :state)
  UNION (SELECT :item, :user, :state, NOW())
  LIMIT 1;

-- After: a plain upsert
INSERT INTO progress_state (item_id, user_id, state, updated_at)
VALUES (:item, :user, :state, NOW()) AS new
ON DUPLICATE KEY UPDATE state = new.state, updated_at = new.updated_at;
```

- Foreign keys and indexes restored. Results compared against a restored 5.7 backup before deploying.
- Upsert deployed to every code path using the old pattern. Failures went from 5 the day before to **0** after deploy.
- A backfill script for the affected completions, run off-hours with before/after counts.
- Follow-ups: error handling and logging around the sync write, and a check for the same pattern in other tenants' history.

## What the workflow contributed

- **Data before theories.** The hourly and daily breakdown, written by the agent and run by me, turned "it's intermittent" into a precise signature. That signature is what pointed at human office-hours activity rather than load.
- **The agent never touched production.** Every diagnostic and the backfill went through review. The lock timeout surfaced *because* the backfill ran as a small, reviewed test first rather than in bulk.
- **One investigation doc from the first hour.** It had a timeline, hypotheses and their status, queries, and affected-user counts, with a "quick start" section at the top, so any session (or teammate) could pick it up without reading chat history.
- **Reusable lesson recorded.** "Self-referential REPLACE plus MySQL 8 means contention; check foreign keys after schema maintenance" went into the project memory. The same root cause turned up shortly after in a related approval flow.
