# Enterprise package (`rulebook-enterprise`)

An Effortless **package seed** (kind `child`). Installing it installs its parts,
in order, each into its own folder:

| Part | Mounted at |
|---|---|
| `rulebook-backend` (erb-seed-rulebook-backend) | `backend/` |
| `rulebook-auth` (erb-seed-rulebook-auth) | `auth/` |
| `rulebook-docs` (erb-seed-rulebook-docs) | `reference/` |

and adds this folder, `enterprise/`, for the regulated parts. Like every
add-in it has no rulebook of its own; it reads the project's
`../../effortless-rulebook/effortless-rulebook.json`.

## What this folder adds

| Path | From | What it is |
|---|---|---|
| `migrations/` | postgres-diff-migration | Migration SQL that takes the live database to the next version (Stripe's pg-schema-diff, with hazard warnings). **Disabled step, run on demand**: edit the two connection strings in `effortless.json` (`new_database` = a database built from the new rulebook, `live_database` = production), then `effortless build -id postgres-diff-migration`. Review `migration.sql` before applying it. |
| `conformance/` | rulebook-to-gold-answer-key | The gold answer key: the rulebook with every derived value computed by Postgres, plus `blank-test.json`. Any other substrate (the TypeScript SDK, a Python port) must reproduce it. **Disabled by default** until the published tool can reach rulebook-to-postgres; then `effortless build -id rulebook-to-gold-answer-key`. |
| `audit-policy.json` | this seed (filled from your answers) | Retention period, the audited tables, and the name of the audit table |

## Audit trail

An enterprise project is expected to keep an audit trail **in the rulebook**,
not beside it: a table named `AuditEvents` (`audit-policy.json`
`auditEventsTable`) that the backend's Postgres schema generates like any other
table. No tool generates it for you; add it to the rulebook. Each row is one
change to one row of an audited table:

| Field | Type | Meaning |
|---|---|---|
| `AuditEventId` | raw, text | Primary key |
| `OccurredAt` | raw, datetime | When the change was committed |
| `ActorEmail` | raw, text | The verified email from the sign-in token (`app.jwt()`), never a client-supplied value |
| `ActorRole` | raw, text | The ERB role the change was made under |
| `TableName` | raw, text | The audited table |
| `RowId` | raw, text | The changed row's primary key |
| `Action` | raw, text | `insert`, `update` or `delete` |
| `BeforeJson` / `AfterJson` | raw, text | The row before and after |
| `RetainUntil` | calculated | `OccurredAt` plus `retentionDays` |

Expectations: every insert, update and delete on a table listed in
`auditedTables` (blank means every table) writes one row in the same
transaction; `AuditEvents` is append-only (the Security module grants no role
update or delete on it, and only admin-equivalent roles read it); rows are kept
at least `retentionDays` days (default 2555, seven years).

## Questions

| Key | Default | Used for |
|---|---|---|
| `retentionDays` | `2555` | `audit-policy.json` |
| `auditedTables` | blank (every table) | `audit-policy.json` |

The parts ask their own questions (`postgresSchema`, `magicLinkTenantId`,
`userTables`).
