# ARCHITECTURE.md — Workbase

How the system is put together. Read with `RULES.md` (what must be true), `PRD.md` (what it must do)
and `DESIGN.md` (how it must look).

## 1. Shape

```
Browser
  └─ Next.js app            pages/screens, server components for reads, client components for forms
        │  HTTPS, session cookie
        ▼
  Node.js API (Express)     routes → validation → service → repository
        │  SQL (connection pool, transactions)
        ▼
  MySQL 8                   workbase schema (workbase-schema-mysql.sql)

  Object/file store         employee documents and leave attachments (not in MySQL)
```

One company, one database, one deployment per environment (`dev`, `staging`, `prod`). No
multi-tenant concerns anywhere in the code.

## 2. Layers and their jobs

| Layer | Responsibility | Must not |
|---|---|---|
| **Route** | HTTP shape: parse, validate input, authenticate, authorise, map result to status code | Contain business logic or SQL |
| **Service** | Business rules from `RULES.md`, transactions, audit writes, orchestration | Know about HTTP or SQL dialects |
| **Repository** | Queries and mapping rows to domain objects | Decide anything |
| **Domain helpers** | Pure functions: working-day count, balance maths, employment-type derivation | Touch the database |

A rule belongs in a **service or a pure helper**, never in a route handler, a React component or a
SQL trigger — the one trigger already in the schema (emergency-contact limit) is a safety net, not
the enforcement point.

## 3. Suggested source layout

```
/app            Next.js routes and screens (one folder per module)
  /components   shared UI built from DESIGN.md (header, tabs, card, badge, table, modal, stat tile)
/api
  /routes       one file per resource
  /services     employee, employment, holiday, calendar, leave, approval, user, audit
  /repos        one per aggregate, SQL only
  /domain       pure rule helpers + their unit tests
  /middleware   auth, permissions, error handler, request logger
/db
  /migrations   ordered, forward-only
  schema.sql    mirror of the live schema, updated with each migration
/docs           PRD.md, DESIGN.md, RULES.md, ARCHITECTURE.md, TASK.md, MEMORY.md
/Design         approved static HTML screens (reference only)
```

## 4. Request lifecycle

1. **Session** — cookie → session record → `user` (id, employee_id, role_id, status). A user whose
   status is not `Active` is rejected here.
2. **Permission** — middleware reads `role_permissions` for `(role_id, module)` and checks the
   action the route declares (`view|edit|approve|override|delete`). `roles.is_admin` short-circuits
   to allowed. Denials return 403 and are not leaked as 404s.
3. **Validation** — schema validation at the boundary; errors return field-level messages.
4. **Service** — runs the rules, opens a transaction when more than one table changes, writes the
   audit rows in the same transaction, returns a domain result.
5. **Response** — `{ data }` or `{ error: { code, message, field? } }`. Dates as ISO strings.

## 5. Cross-cutting concerns

### Audit
Audit is not optional and not a separate call the caller may forget. Implement it as a repository
wrapper: mutations go through a helper that takes `(module, record_type, record_id, changes[], actor)`
and writes `audit_change_log` rows inside the caller's transaction. Login, logout and failed login
go to `user_activity_log`. `v_leave_history` and `v_employment_history` serve the read tabs.

### Effective-dated records
`employment_records` and `employee_addresses` are append-only. One service method
(`closeAndInsert`) does both halves: set the current row's `effective_to = new.effective_from − 1
day`, insert the new row, assert no overlap for that employee. Nothing else may write those tables.

### Calendar resolution
A single service answers "is date D a working day for employee E", used by leave counting, the
calendar screens and dashboards:

```
1. Sunday                            → non-working
2. Saturday                          → saturday_calendar.status for that date,
                                       narrowed by scope tables and employee override
3. holiday on that date              → applicable if work_mode_scope matches the employee's
                                       work mode, department scope matches, and no employee
                                       override says otherwise
4. otherwise                         → working day
```
Individual overrides always win over department and work-mode scope. Cache per (year, employee) for
the duration of a request; never cache across requests.

### Leave engine
Pure helpers, independently testable:
`countWorkingDays(range, halfDayType, calendar)` · `checkBalance(type, days, balance)` ·
`checkConsecutiveCap(type, range)` using `leave_types.max_consecutive_days` ·
`applyApproval(application, decision)` which writes `leave_approvals`, updates `leave_balances`
(`balance = opening + earned − used`, capped by `max_balance_cap`), logs an override in
`leave_override_log` when present, and audits — all in one transaction.

### Files
Employee documents and leave attachments go to object storage; the database keeps the URL/key
(`employee_documents`, `leave_applications.attachment_url`). Access is proxied through the API so
permissions apply; never expose a public bucket URL.

### Configuration
All environment-specific values come from env vars: database credentials, session secret, storage
bucket, base URL, mail transport. No secrets in the repo. Timezone (`Asia/Kolkata`) and currency
(`INR`) are config constants, not literals scattered through the code.

### Errors and logging
One error handler at the edge translates domain errors (`NotFound`, `Forbidden`, `RuleViolation`,
`ValidationError`) to status codes. Logs are structured, carry a request id and the actor's user id,
and never contain passwords, full bank account numbers or document contents.

## 6. Data notes that shape the code

- **Salary is `employment_records.monthly_salary`, in INR.** Display as `₹60,000 / month`. (The
  static screens show `$60,000` and `DESIGN.md`/`PRD.md` currently say "annual" — see `MEMORY.md`
  decision D-07; the schema is right.)
- `leave_types` carries the rules as data: `max_consecutive_days` (CL = 2), `requires_balance`,
  `is_paid`, `max_balance_cap` (PL = 45). Read them; never hard-code.
- `holidays.day_of_week` is a generated column — do not write it.
- `users.password_hash` is NULL until an invite is accepted; `failed_login_count` drives the lock at
  5 (see `RULES.md` R-A4).
- `leave_approvals.final_leave_type_id` / `final_days` let an approver approve something different
  from what was requested; the balance update uses the final values, not the requested ones.
- Indexes already exist for the hot paths (`employment_records(employee_id, effective_from)`,
  `leave_applications(employee_id, status)`, `holidays(holiday_date)`, audit by module/record and by
  date). Add an index with the migration that needs it, not speculatively.

## 7. Frontend architecture

- Next.js app router. Reads render on the server where possible; forms are client components.
- One shared component library built from `DESIGN.md` — header, role-based tab bar, card, card
  header, badge, stat tile, table shell, modal, filter bar. Screens compose these; no screen
  re-declares tokens or component CSS.
- Server state through a query layer with cache invalidation on mutation; no global store for data
  the server owns. Local UI state (open modal, selected tab) stays in the component.
- Permissions shape the UI (hide what the user cannot do) but are never the enforcement point.

## 8. Testing strategy

| Level | Covers |
|---|---|
| Unit | Domain helpers: working-day counting, balance maths, consecutive cap, employment-type derivation, calendar resolution |
| Integration | Service + database on a test schema: effective-dating, approval transaction, audit rows written, permission denial |
| End-to-end | The PRD acceptance criteria (§18), happy path per module |

Seed data for tests mirrors `DESIGN.md` §11 (fixed cast, fixed "today") so expected values are stable.

## 9. Environments and deployment

- `dev` (local MySQL), `staging` (production-like data volume, sanitised), `prod`.
- Migrations run on deploy, forward-only; every migration is reversible in effect (write the
  compensating migration rather than editing history).
- Backups: nightly full dump of `workbase`, retained per the policy still to be set (PRD Q10).

## 10. Deliberately out of scope

Multi-company/multi-currency, payroll processing, attendance devices, mobile apps, real-time
push/websockets, and any background job beyond the year-rollover task once entitlement rules are
settled (PRD Q2, Q3).
