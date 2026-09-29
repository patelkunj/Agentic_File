# MEMORY.md — Workbase project memory

Carry-over context for anyone (human or agent) picking this project up cold. Decisions with their
reasons, the current state of things, known conflicts, and the vocabulary used in this project.

**Keep this file current.** When a decision is made, add a row to §2 with the date. When something is
settled that was open, move it out of §5. Do not delete history — supersede it.

---

## 1. Where things stand (29 Sep 2026)

| Artifact | State |
|---|---|
| Screen designs (12 screens) | Approved, in `Design/` |
| Database schema | Written: `workbase-schema-mysql.sql`, MySQL 8, 26 tables + 2 views + 1 trigger |
| Documentation | `PRD.md`, `DESIGN.md`, `RULES.md`, `ARCHITECTURE.md`, `AGENTS.md`, `TASK.md`, this file |
| Application code | **Not started** |
| Open blockers | Leave entitlement rules, notifications, auth method (PRD §16) |

Latest screen files: `login.html`, `admin-dashboard-v3.html`, `user-dashboard-v5.html`, and the nine
module screens as `*-v3.html`. Older versions sit beside them and can be deleted.

## 2. Decision log

| # | Date | Decision | Why / notes |
|---|---|---|---|
| D-01 | Sep 2026 | Name: **Workbase** (also referred to as project EMP) | |
| D-02 | Sep 2026 | **Next.js** frontend, **Node.js** backend, **MySQL 8** | Matches the team's existing skills |
| D-03 | Sep 2026 | Single timezone **IST**, single currency **INR** | No multi-region plans; simplifies dates and money |
| D-04 | Sep 2026 | Employee ≠ User: a login is created only for staff who need access | Most employees never sign in |
| D-05 | Sep 2026 | Employment terms are **effective-dated and append-only** | The original problem: history was being overwritten |
| D-06 | Sep 2026 | **No automatic CL/PL → LOP splitting** of a leave application | The applicant or approver chooses the type explicitly; silent conversion surprised people |
| D-07 | Sep 2026 | Salary is **monthly, in INR** (`employment_records.monthly_salary`) | The `$60,000` in the static screens is a mock artifact; with INR a monthly figure is the sensible reading. Supersedes the "annual" wording in `PRD.md`/`DESIGN.md` |
| D-08 | Sep 2026 | **Saturday Off Calendar** is the single source of truth for Saturday working status | Prevents per-employee Saturday drift |
| D-09 | Sep 2026 | Sunday is always non-working; only Sunday holidays are auto-flagged as "weekend" | Saturdays vary, so flagging them would be wrong |
| D-10 | Sep 2026 | Leave codes are **CL, PL, LOP** only | "CSL" appeared in some screens; it is not a real code |
| D-11 | Sep 2026 | Rule parameters live in `leave_types` as **data** (CL cap 2, PL cap 45) | Changing a cap must not need a deploy |
| D-12 | Sep 2026 | Audit is **read-only to everyone, including Admin** | The log is worthless if it can be edited |
| D-13 | Sep 2026 | Top header with logo left, user menu right (Profile / Change Password / Logout), on every screen except login | |
| D-14 | Sep 2026 | Navigation is **role-based**: 7 admin tabs, 3 employee tabs, as real links | Employees were seeing admin modules |
| D-15 | Sep 2026 | Login page carries **no branding text**, just the sign-in card | Deliberate minimal choice |
| D-16 | Sep 2026 | Mock "today" is **16 Sep 2026**; all sample data is 2026 and internally consistent | Screens are shown to stakeholders |

## 3. Known conflicts to clean up

These are documented rather than silently fixed, because they touch several files.

1. **Salary wording.** `PRD.md` (§13, FR-4.2), `DESIGN.md` §11 and `AGENTS.md` §6 say *annual*; the
   schema says `monthly_salary` and D-07 settles it as **monthly INR**. The static screens show
   `$60,000`. Fix: docs → monthly, screens → `₹60,000 / month`. *(Task T-0.4)*
2. **Auto-LOP split.** `PRD.md` FR-6.7 describes automatic splitting; D-06 rejects it. `RULES.md`
   R-L7 is correct. Fix: remove FR-6.7 or rewrite it as "no automatic splitting".
3. **"CSL"** still appears in warnings on Leave Application, Leave Approval and Audit, and in a
   schema comment. Correct code is **CL** (D-10).
4. **Locale mix.** Static screens use US addresses and phone numbers alongside Indian tax fields
   (TDS, PAN) and Indian holidays. D-03 settles it: INR/IST, Indian formats.
5. **Old UI patterns** in the pre-v3 screens: `<div onclick>`, unlabelled inputs, 9–10px text.
   Not to be copied into the app.

## 4. Things that surprised us (worth remembering)

- The leave, holiday and Saturday rules interact: a day count depends on the holiday scope rules
  *and* the Saturday calendar *and* individual overrides. Build the calendar resolution service once
  (`ARCHITECTURE.md` §5) and let everything else call it.
- Several sample people (Rita Sharma, Marcus Tan, Priya Nair) have **no login**, so their leave is
  entered by HR on their behalf. The audit trail has to express "on behalf of" or the data looks
  wrong.
- Approvers can change both the leave type and the day count at review, so balance updates must use
  the final values, not the requested ones (`leave_approvals.final_*`).
- Dashboard tiles read from every module, so they are the last thing to build, not the first.

## 5. Open questions (mirror of `PRD.md` §16)

| # | Question | Blocks |
|---|---|---|
| Q2 | Entitlements: fixed per year, accrued monthly, or pro-rated for joiners? | T-6.1 |
| Q3 | Year-end carry-over and cap behaviour | T-6.1, rollover job |
| Q4 | Comp-off expiry; redeemable as leave or paid out? | T-5.7 |
| Q5 | Email notifications in scope (invite, applied, approved, rejected)? | T-2.4, T-6.x |
| Q6 | Auth: local password only, or SSO (the login screen shows an SSO button)? | T-2.1 |
| Q7 | Can an employee withdraw or amend a submitted request? | T-6.9 |
| Q8 | Who may see salaries — HR only, or Reporting Managers too? Permissions are module-level today | T-2.3 |
| Q9 | Probation affecting leave eligibility? | T-6.1 |
| Q10 | Audit retention and backup policy | T-9.5 |

*(Q1 locale is closed by D-03.)*

## 6. Glossary

| Term | Meaning |
|---|---|
| **CL** | Casual leave. Paid, capped at 2 consecutive days without override |
| **PL** | Privilege / earned leave. Paid, balance capped at 45 |
| **LOP** | Loss of pay. Unpaid, no balance, no cap |
| **RH** | Restricted holiday — optional, drawn from the employee's RH allowance |
| **Comp-off** | A credited day earned by working a holiday or an `Off` Saturday |
| **Effective-dated record** | A row valid from a date until superseded; never edited afterwards |
| **Override** | Approving something that breaks a rule, with a recorded reason |
| **Work mode** | Remote · In House · Hybrid |
| **Scope / applicability** | Which employees a holiday or Saturday rule applies to |
| **On behalf of** | An action taken by HR/Admin for an employee who has no login |

## 7. People in the sample data

Admin User (System Administrator) · Employee A (EMP-00214, Senior Developer, Engineering) ·
Sarah Mehta (HR) · J. Whitfield (Reporting Manager, Engineering) · Rita Sharma (Product Designer) ·
Marcus Tan (DevOps Engineer, invited, never logged in) · Priya Nair (QA Analyst) ·
Tom Nguyen (locked account, 5 failed logins). Keep these names in seeds and tests.

## 8. How to update this file

- A decision is made → new row in §2, dated, with the reason. Never edit an old row; add a
  superseding one and say which number it supersedes.
- A conflict in §3 is fixed → strike it from §3 and note the fix in §2 if it changed a decision.
- An open question is answered → move it from §5 into §2 and update `PRD.md` §16.
- Something surprising bites you → §4, so the next person doesn't lose the same hour.
