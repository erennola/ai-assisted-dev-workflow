# CLAUDE.md — Acme Learning (example)

<!--
  Example project memory for a FICTIONAL app ("Acme Learning": PHP 8 backend,
  React frontend, MySQL 8, one database per tenant). Claude Code loads this file
  automatically at the start of every session, so it should be:
    - short (aim for under ~150 lines; every line costs context on every session),
    - about THIS repo (stack, commands, traps), not general advice,
    - a map to deeper docs, not a copy of them.
  Comments like this one explain the choices and can be deleted.
-->

## Read first
- `docs/WORKFLOW.md`: how we work (roles, production rules, ticket hygiene). It overrides anything below.
- `kb/README.md`: knowledge-base entry point. Open the folder `INDEX.md` you need. Never load whole folders.

## Stack
- Backend: PHP 8.2, custom MVC in `app/`, modules in `app/Modules/<Name>/`.
- Frontend: React 18 + TypeScript in `web/`, built with Vite into `public/build/`.
- Data: MySQL 8.0. **One database per tenant** (`tenant_<slug>`). The same codebase serves every tenant, so a code change ships to all of them.
- Infra: AWS (EC2 + RDS + S3). Local runs on Docker Compose.

<!-- The per-tenant line is the single most important fact here: it changes how every
     query and every code change should be reasoned about. Put facts like that first. -->

## Commands
```bash
docker compose up -d                          # app on http://localhost:8000, mysql on :3307
docker compose exec app composer test         # PHPUnit
docker compose exec app composer lint         # PHP-CS-Fixer + PHPStan (level 6)
npm --prefix web run test                     # Vitest
npm --prefix web run build                    # production bundle
./scripts/new-tenant.sh <slug>                # local tenant DB + vhost
./scripts/import-dump.sh <slug> <file.sql.gz> # load a sanitised dump into a local tenant
```

## Rules (short form; details in WORKFLOW.md)
- Production and staging: write SQL for me to run. Never execute it yourself.
- Local Docker: OK to run queries and changes, but say what you're about to run first.
- Check the schema before writing SQL: `database/migrations/` and `app/Models/`.
- Agree the approach before changing code. Present alternatives.
- Commit on a branch named `<TICKET>-<desc>`. Don't push.
- Jira/Linear: comment when asked. Never transition, re-assign or re-prioritise without my explicit OK.
- English for code, commits, PRs and docs.

## Domain map
- Enrollment → progress tracking → completion → certificate is the core flow.
  Code path: `Modules/Courses/Progress.php` → `Modules/Awards/AwardService.php` → `Modules/Awards/CertificateIssuer.php`.
- Nightly jobs: `app/Jobs/` (scheduled from `config/schedule.php`). The reset job (`ResetExpiredAwards`) is the riskiest code in the repo, so treat changes there as high-risk.
- Reports read from replicas via `ReportDb`. Never write through it.

## Known traps
<!-- Each bullet is a lesson that cost real time. One line, plus a link to the write-up.
     Add only when a ticket proves the trap is reusable, not for every bug. -->
- Completion rows survive an award reset. A re-enabled award can auto-grant old completions. See `kb/incidents/awards/sup-123-certificates-after-retake.md`.
- MySQL 8 strict mode: zero dates (`0000-00-00`) in legacy rows break `INSERT ... SELECT`. Filter or coalesce them.
- Mixed collations between tenant DBs break cross-DB joins in reports. Use `COLLATE utf8mb4_unicode_ci` explicitly.
- The tenant slug is not always the DB name (renamed or merged tenants). Confirm with `SHOW DATABASES LIKE '%<hint>%'`.
- A recurring symptom is not proof of one recurring bug. Check the current cause in current data.

## Definition of done
Fix merged or diagnosis delivered **and** an incident write-up in `kb/incidents/`, plus one line in its `INDEX.md`.
