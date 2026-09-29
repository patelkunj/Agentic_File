# TASK.md — Workbase build backlog

Working backlog for agents and humans. Tick a box only when the task's checks have actually run.
Task IDs are stable; reference them in commits (`leave: day counting (T-4.2, FR-6.5)`).

**Status at last update (29 Sep 2026):** design and documentation complete; no application code yet.

Legend: `[ ]` not started · `[~]` in progress · `[x]` done · `⛔` blocked (reason in the row)

---

## Phase 0 — Groundwork

- [x] **T-0.1** Screen designs for all 12 screens (`Design/*.html`)
- [x] **T-0.2** MySQL schema (`workbase-schema-mysql.sql`)
- [x] **T-0.3** Product docs: `PRD.md`, `DESIGN.md`, `AGENTS.md`, `ARCHITECTURE.md`, `RULES.md`
- [ ] **T-0.4** Fix known doc conflicts: salary is monthly INR (not annual), remove auto-LOP split
      from PRD FR-6.7, correct "CSL" → "CL", set locale to IST/INR throughout — see `MEMORY.md` §3
- [ ] **T-0.5** Answer the open questions in `PRD.md` §16 (at minimum Q2 entitlements, Q3 carry-over,
      Q5 notifications, Q6 auth) ⛔ *needs Kunj's decisions; blocks T-4.1 and T-6.x*
- [ ] **T-0.6** Repo skeleton: Next.js app, Node API, folder layout per `ARCHITECTURE.md` §3
- [ ] **T-0.7** Tooling: TypeScript, lint, format, test runner, CI running lint + typecheck + tests
- [ ] **T-0.8** Database connection pool, migration runner, `.env.example`, local dev instructions
- [ ] **T-0.9** Seed script: departments, roles, leave types, the `DESIGN.md` §11 sample cast

## Phase 1 — Shared foundation

- [ ] **T-1.1** UI kit from `DESIGN.md`: tokens, header, role-based tab bar, card + card header,
      badge set, stat tile, table shell, modal, filter bar, banner set
- [ ] **T-1.2** App shell: page layout, header with Profile / Change Password / Logout, role-based tabs
- [ ] **T-1.3** Responsive pass on the kit (390px rules from `DESIGN.md` §9)
- [ ] **T-1.4** API scaffolding: route → service → repo wiring, validation helper, error handler,
      `{data}` / `{error}` envelope
- [ ] **T-1.5** Audit helper — mutations write `audit_change_log` inside the caller's transaction (R-D1)
- [ ] **T-1.6** Domain helpers + unit tests: working-day counting, balance maths, employment-type
      derivation, effective-date close/insert

## Phase 2 — User & Access (M1)

- [ ] **T-2.1** Login: form, session cookie, password hashing, `Invited` → `Active` on first login (R-A5)
- [ ] **T-2.2** Failed-login counter, lock at 5, Admin unlock, both audited (R-A4–R-A7, R-D2)
- [ ] **T-2.3** Permission middleware from `role_permissions`, Admin short-circuit (R-A3, R-A4)
- [ ] **T-2.4** User Management screen: list, filters, invite, change role, reset password, disable
- [ ] **T-2.5** Role & Permission matrix screen: per-role editing, Admin row locked (FR-2.x)
- [ ] **T-2.6** Change password from the header menu (FR-8.5)
- [ ] **T-2.7** Tests: permission denial (403), lock flow, Admin immutability, last-Admin guard

## Phase 3 — Employee Management (M2)

- [ ] **T-3.1** Employee list and creation
- [ ] **T-3.2** Profile — Personal tab (identity fields, company-owned fields read-only) (R-E1, R-E6)
- [ ] **T-3.3** Address history with effective dating (R-E2)
- [ ] **T-3.4** Emergency contacts, max 2 with a clean error (R-E3)
- [ ] **T-3.5** Documents: upload to object storage, Admin approve/reject with reason (R-E4)
- [ ] **T-3.6** Bank details with masking everywhere and an audited export path (R-E5)
- [ ] **T-3.7** Rejoin linking and tenure spanning both records (R-E7, R-J10)

## Phase 4 — Employment & Job History (M3)

- [ ] **T-4.1** `closeAndInsert` service: append-only records, no overlaps (R-J1–R-J3)
- [ ] **T-4.2** Add/Edit Job Record screen: sections, carry-forward, diff summary before save (R-J4)
- [ ] **T-4.3** Work schedule editor; net hours and Full/Part-Time derivation with override (R-J6, R-J7)
- [ ] **T-4.4** Saturday status shown read-only from the Saturday calendar (R-J8)
- [ ] **T-4.5** Professional engagement fields (R-J9)
- [ ] **T-4.6** Timeline view with position tenure, overall tenure, change-type badges
- [ ] **T-4.7** Tests: AC-1, overlap rejection, type derivation at the 7-hour boundary

## Phase 5 — Holidays & Calendar (M4)

- [ ] **T-5.1** Holiday CRUD with type and recurring flag (R-H1)
- [ ] **T-5.2** Applicability scoping: work mode, department, individual overrides (R-H2)
- [ ] **T-5.3** Calendar resolution service — "is D a working day for E" (R-H3, `ARCHITECTURE.md` §5)
- [ ] **T-5.4** Saturday Off Calendar: per-date status, scopes, overrides (R-H6)
- [ ] **T-5.5** Quick pattern generator that never overwrites manual entries (R-H7)
- [ ] **T-5.6** Copy previous year with skip-by-name and a review prompt for festival dates (R-H8)
- [ ] **T-5.7** Comp-off credit and redemption (R-H9)
- [ ] **T-5.8** Employee-facing calendar (month grid + list, filters)
- [ ] **T-5.9** Tests: resolution order, pattern non-overwrite, AC-3

## Phase 6 — Leave (M5)

- [ ] **T-6.1** Leave types and balances seeded per employee per year ⛔ *depends on T-0.5 Q2/Q3*
- [ ] **T-6.2** Apply for Leave screen: types, dates, half day, reason, attachment (R-L4)
- [ ] **T-6.3** Day counting against the calendar service, half days (R-L5)
- [ ] **T-6.4** Balance preview before submit; no automatic splitting (R-L6, R-L7)
- [ ] **T-6.5** Consecutive-cap flagging from `leave_types` (R-L8)
- [ ] **T-6.6** Approver routing from the current reporting manager (R-L9)
- [ ] **T-6.7** Draft save and submit (R-L10); HR submission on behalf (R-L11)
- [ ] **T-6.8** Approval queue: status tabs, filters, rule-breach flags, balance-after column
- [ ] **T-6.9** Review modal: type conversion, day reduction, approve / reject / override (R-L13–R-L15)
- [ ] **T-6.10** Approval transaction: balances + approval row + override log + audit (R-L16)
- [ ] **T-6.11** Self-approval guard (R-L18)
- [ ] **T-6.12** Leave statistics view (used vs remaining per type, LOP, comp-off)
- [ ] **T-6.13** Tests: AC-2, AC-4, AC-5, R-L7, transaction rollback

## Phase 7 — Audit & History (M6)

- [ ] **T-7.1** Change History tab with before → after diffs
- [ ] **T-7.2** User Activity tab (logins, failures, creates, edits, deletes)
- [ ] **T-7.3** Leave History tab from `v_leave_history`, overrides highlighted with reasons
- [ ] **T-7.4** Employment History tab from `v_employment_history`
- [ ] **T-7.5** Filters (module, actor, action, date range) and CSV export
- [ ] **T-7.6** Test: audit is read-only through every path (R-D3), AC-6

## Phase 8 — Dashboards (M7)

- [ ] **T-8.1** Employee dashboard: balances, tenure, pending count, own requests, upcoming holidays,
      quick actions, calendar modal (FR-9.1)
- [ ] **T-8.2** Admin dashboard: personal block plus org stats, pending queue preview, activity feed,
      admin actions (FR-9.2)
- [ ] **T-8.3** Every count links through to the owning screen (FR-9.3)
- [ ] **T-8.4** Performance check: dashboards under 2s with 500 employees (NFR-7)

## Phase 9 — Hardening and release

- [ ] **T-9.1** Accessibility pass: labels, keyboard paths, focus, contrast (NFR-4)
- [ ] **T-9.2** Responsive pass on every screen at 390px
- [ ] **T-9.3** Security review: server-side permission coverage, masking, session handling, rate limits
- [ ] **T-9.4** Load a realistic data set; index review on the hot queries
- [ ] **T-9.5** Backup and restore rehearsal ⛔ *depends on the retention policy, PRD Q10*
- [ ] **T-9.6** UAT against the PRD acceptance criteria (§18)
- [ ] **T-9.7** Deployment runbook and rollback steps

---

## Definition of done (every task)

1. The rule IDs it implements are named in the PR.
2. Server-side enforcement exists where a rule is involved — not just UI.
3. Audit rows are written for every mutation it introduces.
4. Unit or integration tests cover the rule, including its failure path.
5. Lint, typecheck and tests were run, and the PR says so.
6. The UI matches `DESIGN.md` and works at 390px.
7. No new open question left undocumented — add it to `PRD.md` §16 and `MEMORY.md`.
