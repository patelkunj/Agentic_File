# Product Requirements Document — Workbase (Employee Management System)

**Status:** Draft v1 · derived from the approved screen designs in `Design/`
**Owner:** Kunj
**Last updated:** 27 Sep 2026
**Companion docs:** `DESIGN.md` (visual and interaction rules) · `workbase-schema-mysql.sql` (database)

> This PRD is reverse-engineered from the 12 designed screens. Everything in §4–§10 that the screens
> show is stated as a requirement. Everything they do not settle is listed in §13 as an open
> question rather than invented here.

---

## 1. Summary

Workbase is an internal employee management system for a single company. It holds the record of who
works here, what their employment terms are and how those terms changed over time, and it runs the
day-to-day HR loop: leave requests and approvals, the holiday calendar, and an audit trail of every
change.

It is not a payroll system and not a recruiting system. It is the system of record that payroll and
managers read from.

## 2. Problem

Employee data lives in spreadsheets and email threads. Three things break repeatedly:

1. **History is overwritten.** When someone is promoted or gets a raise, the old value disappears, so
   "what was this person's salary in March" cannot be answered.
2. **Leave is manual.** Balances are tracked by hand, rule breaches (too many consecutive casual
   days, negative balance) are caught late or not at all, and there is no record of who approved what.
3. **Nothing is accountable.** There is no log of who changed a field, when, or why.

## 3. Goals

| # | Goal | How we will know |
|---|---|---|
| G1 | Employment history is never lost | Every position, salary and schedule change is a new effective-dated record; no record is edited in place |
| G2 | Leave runs end to end in the system | An employee can apply, see their balance impact before submitting, and an approver can act with the rules shown to them |
| G3 | Rules are enforced, not remembered | Balance checks, the consecutive-CL cap and the holiday/weekend exclusions are applied by the system, and breaking one requires an explicit override with a reason |
| G4 | Every change is attributable | Any field change, login and leave action appears in the audit log with actor, timestamp, before and after |
| G5 | Access matches the role | A permission matrix per role controls view/edit/approve/override/delete per module |

**Non-goals for v1:** payroll runs and payslips, attendance/time clocking, recruitment and onboarding
workflows, performance reviews, expense claims, a mobile app.

## 4. Users and roles

| Role | Who they are | What they do |
|---|---|---|
| **Admin** | System owner (HR/ops lead) | Everything. All permissions locked ON and not grantable away |
| **HR** | HR staff | Maintains employee records, holidays, users; approves leave where granted |
| **Reporting Manager** | Team lead | Sees their people; approves leave where granted |
| **Clerk** | Back-office | Data entry on holidays and records; no approvals |
| **Employee** | Everyone else | Own profile, own leave, holiday calendar |

Two important distinctions the design makes:

- **Employee ≠ User.** An employee record exists for everyone. A *user* is a login account linked to
  an employee record, created only for staff who need system access (FR-1.1).
- **Permission grants are per role, per module, per action.** The actions are View, Edit, Approve,
  Override, Delete. Admin's row is locked; other roles are editable (FR-2.x).

## 5. Scope — modules

| Module | Screens |
|---|---|
| M1 User & Access | User Management, Role & Permission Management |
| M2 Employee Management | Employee Profile (Personal, Documents, Bank, Employment tabs) |
| M3 Employment & Job History | Timeline view, Add/Edit Job Record |
| M4 Holiday Management | Holiday Calendar, Saturday Off Calendar, Comp-Off |
| M5 Leave Management | Apply for Leave, Leave Approval, leave statistics |
| M6 Audit & History | Change History, User Activity, Leave History, Employment History |
| M7 Dashboards | Admin dashboard, Employee dashboard |
| M8 Access | Login |

---

## 6. M1 — User & Access

**FR-1.1** A user account is created by linking an existing employee record to an email address and a
role. Employees without a login are valid and normal.
**FR-1.2** User statuses: Active, Invited (pending first login), Locked, Disabled. Locked is set
automatically after 5 consecutive failed logins (FR-8.3).
**FR-1.3** Admin/HR can invite a user, change a user's role, reset a password, and disable a user.
The user list shows user, linked employee, role, status and last login.
**FR-1.4** A disabled user keeps their employee record and history; only access is removed.
**FR-2.1** Permissions are a matrix of role × module × action (View, Edit, Approve, Override, Delete).
**FR-2.2** Admin has every permission, locked ON, and cannot be edited or removed.
**FR-2.3** Changing a role's permissions takes effect for every user holding that role.
**FR-2.4** Each role shows its user count, so an admin can see the blast radius before editing.

## 7. M2 — Employee Management

**FR-3.1** An employee record holds: full name, employee ID, date of birth, gender, marital status,
personal phone, personal email, company email (assigned by the company), department, position and
status (Active / Inactive / Rejoined).
**FR-3.2** **Address history** is effective-dated. Types: Current, Permanent, Previous. Adding a new
current address ends the previous one — earlier addresses stay visible.
**FR-3.3** **Emergency contacts**: maximum 2 per employee, each with name, relationship and phone.
**FR-3.4** **Documents**: uploaded by the employee, reviewed by Admin. Status is Pending, Approved or
Rejected; only Admin can approve or reject.
**FR-3.5** **Bank details**: entered by the employee; account number is masked on screen and available
in full only in a secure export.
**FR-3.6** A rejoining employee keeps a link to their previous employee ID, and the profile shows the
rejoin marker with both IDs.
**FR-3.7** The profile is read-only for the employee on company-controlled fields (position,
department, salary, company email); those change only through M3.

## 8. M3 — Employment & Job History

**FR-4.1** Employment terms are stored as **effective-dated records**. Saving never edits the current
record: it creates a new record effective from a chosen date and sets the previous record's
"effective to" as one day earlier.
**FR-4.2** A record holds: effective from, change reason (Promotion, Salary Revision, Transfer,
Schedule Change, Rejoin …), position, department, reporting manager, work mode (Remote / In House /
Hybrid), annual salary, work schedule, and responsibilities.
**FR-4.3** Unchanged fields carry forward from the previous record, and the save screen shows a diff
summary ("Effective Date → 1 Oct 2026", "Salary unchanged") before saving.
**FR-4.4** **Work schedule** is per weekday: off flag, start time, end time, plus lunch and tea break
minutes. Daily net hours = end − start − breaks.
**FR-4.5** The system derives **Full-Time / Part-Time** from net daily hours against a 7-hour
threshold, with a manual override that requires a reason.
**FR-4.6** Saturday working status is **not set per employee here**. It comes from the Saturday Off
Calendar (FR-5.4) and is reflected read-only in the schedule.
**FR-4.7** A **professional engagement** flag marks non-salaried contractors: fees are subject to
withholding tax, with a tax section, rate and tax ID captured.
**FR-4.8** The timeline shows every record with position, effective range, salary, schedule and change
type, and calculates position tenure and overall tenure (overall tenure spans rejoins).

## 9. M4 — Holiday Management

**FR-5.1** A holiday has a name, date, auto-derived day of week, type (**Mandatory**, **Optional**,
**Restricted / RH**) and a recurring-yearly flag.
**FR-5.2** **Applicability** can be narrowed from the default "All" by work mode, by department, or by
named individuals (individual overrides win over department and work-mode rules).
**FR-5.3** Holidays falling on a **Sunday** are flagged in the list, because they may warrant a
comp-off. Saturday is not auto-flagged (see FR-5.4).
**FR-5.4** The **Saturday Off Calendar** is the single source of truth for Saturday status. Every
Saturday of the year is a record: Off (acts as a holiday) or Working, with the same applicability
scoping and individual overrides. A "quick pattern" (e.g. all Saturdays working, alternate Saturdays
off) seeds unset Saturdays only and never overwrites ones already edited.
**FR-5.5** **Comp-off**: working on a holiday or an Off Saturday can be credited as a comp-off day,
recorded against the employee with the source date, hours worked and a note.
**FR-5.6** **Copy previous year** clones holiday names, types and applicability into a new year.
Fixed-date holidays shift automatically; festival dates must be reviewed after copying. Existing
holidays with the same name are skipped, not duplicated, and everything stays editable afterwards.
**FR-5.7** Employees see the calendar read-only, by month, with the year's holidays listed and
filterable by type.

## 10. M5 — Leave Management

### Types and balances

**FR-6.1** Leave types are **CL** (casual), **PL** (privilege/earned) and **LOP** (loss of pay,
unpaid, no balance limit). No other codes are used anywhere in the product.
**FR-6.2** Each employee has a per-year entitlement per type; the system tracks used and remaining.
**FR-6.3** Optional and Restricted (RH) holidays taken count against the employee's RH allowance.

### Applying

**FR-6.4** An employee applies with: leave type, from and to dates, duration (full day, first half or
second half — half day only for a single-day request), reason, and an optional supporting document.
**FR-6.5** The day count **excludes weekends and holidays** inside the range.
**FR-6.6** Before submitting, the employee sees the balance impact: current balance, this request,
remaining after approval, with a negative result shown in red.
**FR-6.7** If CL or PL runs out mid-request, the remaining days are automatically applied as **LOP**.
**FR-6.8** **CL is capped at 2 consecutive days.** A longer request is accepted but flagged as
requiring an override at approval.
**FR-6.9** The request is routed automatically to the approver derived from the reporting line, and
the employee is shown who that is before submitting. A request can be saved as a draft.
**FR-6.10** HR may enter a request on behalf of an employee who has no login; the audit record shows
who entered it.

### Approving

**FR-6.11** Admin can approve or reject any request. HR and Reporting Managers can act only where
their role has been granted approval permission for that module.
**FR-6.12** The approval queue is grouped by status (Pending, Approved, Rejected, All), filterable by
leave type and department, and shows the balance after approval for each row.
**FR-6.13** Rule breaches are flagged in the queue itself — exceeding the CL cap, or a request with
no balance left.
**FR-6.14** On review the approver can **convert the leave type** (e.g. CL → LOP when balance is
insufficient) and **reduce the day count** before approving.
**FR-6.15** Approving something that breaks a rule requires ticking the override box and entering an
**override reason**; approver and date are recorded. Rejection also requires a reason.
**FR-6.16** Approved leave updates the employee's balance immediately and appears in leave history.
**FR-6.17** A statistics view shows used vs remaining per type for every employee, including LOP taken
and comp-off balance.

## 11. M6 — Audit & History

**FR-7.1** The audit log is **read-only for everyone, including Admin**, and every module writes to it.
**FR-7.2** Four views: **Change History** (field-level before → after), **User Activity** (logins,
failed logins, creates, edits, deletes), **Leave History** (applied, approved, rejected, overridden,
with reasons), **Employment History** (effective-dated employment changes).
**FR-7.3** Each entry records timestamp, module, field, old value, new value, and the actor with their
role — including "on behalf of" entries.
**FR-7.4** Entries are filterable by module, actor, action type and date range, and exportable as CSV.
**FR-7.5** Overrides are highlighted, and the override reason is shown inline.

## 12. M7/M8 — Dashboards and access

**FR-9.1** **Employee dashboard**: own context strip, CL and PL remaining, position tenure, pending
request count; own recent leave requests with status; upcoming holidays; quick actions (apply for
leave, profile, documents, leave history); full holiday calendar in a modal.
**FR-9.2** **Admin dashboard**: everything in FR-9.1 for the admin's own record, plus an organisation
section — headcount with new joiners this month, pending leave requests, who is on leave today, open
documents — plus a pending-approval queue preview, a recent-activity feed, and admin quick actions.
**FR-9.3** Counts and lists on a dashboard link through to the screen that owns them.
**FR-8.1** **Login** takes username or email and password, with remember-me and forgot-password.
**FR-8.2** Accounts are provisioned by HR/Admin; there is no self sign-up.
**FR-8.3** Five consecutive failed attempts lock the account; unlocking is an Admin action.
**FR-8.4** Failed and successful logins are written to the audit log.
**FR-8.5** Signed-in users can change their own password from the header menu.

---

## 13. Data model (as implied by the screens)

Confirm against `workbase-schema-mysql.sql` before building.

```
employee            id, employee_code, name, dob, gender, marital_status,
                    personal_phone, personal_email, company_email, status,
                    previous_employee_id (rejoin), created_at
employee_address    employee_id, type(current|permanent|previous), address, from, to
emergency_contact   employee_id, name, relationship, phone           (max 2)
employee_document   employee_id, kind, file, status(pending|approved|rejected), uploaded_at, reviewed_by
bank_detail         employee_id, holder_name, bank_name, account_no(masked), ifsc_routing
user                employee_id, email, role_id, status(active|invited|locked|disabled), last_login_at
role                name, is_system(admin locked)
role_permission     role_id, module, view, edit, approve, override, delete
job_record          employee_id, effective_from, effective_to, change_reason, position,
                    department, reporting_manager_id, work_mode, annual_salary,
                    is_professional, tax_section, tax_rate, tax_id, responsibilities
work_schedule       job_record_id, weekday, is_off, start_time, end_time,
                    lunch_break_min, tea_break_min, employment_type_override, override_reason
holiday             year, name, date, type(mandatory|optional|restricted), recurring
holiday_scope       holiday_id, work_mode, department_id, employee_id   (individual wins)
saturday_status     date, status(off|working), scope…                  (source of truth)
comp_off            employee_id, source_date, hours, note, credited_by
leave_balance       employee_id, year, type(CL|PL), entitled, used
leave_request       employee_id, type, from, to, duration(full|first_half|second_half),
                    days, reason, document, status(draft|applied|approved|rejected),
                    approver_id, decided_at, override_flag, override_reason, entered_by
audit_log           at, actor_id, on_behalf_of, module, entity, field, old_value, new_value, action
```

## 14. Non-functional requirements

- **NFR-1 Design system.** Every screen follows `DESIGN.md`: tokens, components, role-based tabs,
  header with profile / change password / logout.
- **NFR-2 Browsers.** Latest Chrome, Edge, Firefox and Safari.
- **NFR-3 Responsive.** Usable down to 390px: tabs scroll, forms stack, wide tables scroll inside
  their card. Desktop is the primary target.
- **NFR-4 Accessibility.** Labels tied to inputs, keyboard-reachable controls, visible focus ring,
  11px minimum readable text, status never conveyed by colour alone (badges carry text).
- **NFR-5 Security.** Passwords hashed; account lockout per FR-8.3; permission checks enforced
  server-side, not only in the UI; bank account numbers masked in all screen output; audit entries
  immutable.
- **NFR-6 Data integrity.** Effective-dated records never overlap for one employee; approving leave
  and writing its audit entry happen in one transaction.
- **NFR-7 Performance.** List screens page or lazy-load beyond ~200 rows; dashboards load under 2s
  with 500 employees.
- **NFR-8 Timezone and locale.** One company timezone and one locale for currency, dates, phone and
  tax fields — see the open question below.

## 15. Assumptions

1. One company, one timezone, one currency; no multi-tenant or multi-entity support.
2. Salary is stored as an **annual** figure.
3. Leave entitlements are per calendar year and set per employee.
4. The approver is derived from the reporting manager on the current job record.
5. Sunday is always a non-working day; Saturday is governed by the Saturday Off Calendar.

## 16. Open questions

| # | Question | Why it matters |
|---|---|---|
| Q1 | Which country/locale? The designs mix US addresses and phone numbers with Indian tax (TDS/PAN) and Indian holidays | Decides currency, tax fields, date format and the holiday seed list |
| Q2 | How are leave entitlements set — fixed per year, accrued monthly, pro-rated for joiners? | Changes the balance engine |
| Q3 | Do CL/PL balances carry over at year end, and is there a cap? | Year-rollover job |
| Q4 | Do comp-off days expire, and can they be taken as leave or only paid out? | Comp-off flow is only sketched in the designs |
| Q5 | Are email notifications in scope (invite, applied, approved, rejected)? No screen covers them | Affects M1 and M5 |
| Q6 | Authentication: local password only, or SSO? The login screen shows an SSO option | Auth build |
| Q7 | Can a rejected or approved request be withdrawn or amended, and by whom? | Missing state transitions |
| Q8 | Who may see other employees' salaries — HR only, or Reporting Managers too? | Permission matrix granularity (module-level today) |
| Q9 | Is there a probation period affecting leave eligibility? | Leave rules |
| Q10 | Retention/backup policy for audit entries | NFR-5 |

## 17. Release plan (suggested)

- **Phase 1 — Records.** M1 (users, roles), M2 (employee profile), M6 write path. The system becomes
  the record of truth.
- **Phase 2 — Employment.** M3 effective-dated job records and work schedules, Employment History view.
- **Phase 3 — Calendar.** M4 holidays, Saturday Off Calendar, comp-off.
- **Phase 4 — Leave.** M5 apply, approve, balances, rules and overrides, Leave History.
- **Phase 5 — Dashboards.** M7 admin and employee dashboards once the data behind the tiles exists.

## 18. Acceptance criteria (samples)

- **AC-1** Changing an employee's salary creates a second job record; the first is still visible in
  the timeline with an "effective to" one day before the new one.
- **AC-2** A 3-day CL request shows "requires override" in the approval queue, and approving it
  without a reason is refused.
- **AC-3** A leave request spanning a Saturday marked Off and a mandatory holiday counts neither day.
- **AC-4** An employee with 1 CL remaining applying for 3 CL days gets 1 CL + 2 LOP, shown before
  submitting.
- **AC-5** An HR user without Approve permission on Leave Management sees the queue but no action
  buttons.
- **AC-6** Every change in AC-1 to AC-5 appears in the audit log with actor, timestamp and
  before/after values.

## 19. Screen inventory

| Screen | File |
|---|---|
| Login | `login.html` |
| Admin dashboard | `admin-dashboard-v3.html` |
| Employee dashboard | `user-dashboard-v5.html` |
| User Management | `User-Management-v3.html` |
| Role & Permission Management | `Role-Permission-Management-v3.html` |
| Employee Profile | `Employee-Profile-v3.html` |
| Employment & Job History | `Employment-Job-History-v3.html` |
| Add / Edit Job Record | `Add-Edit-Job-Record-v3.html` |
| Holiday Management | `Holiday-Management-v3.html` |
| Apply for Leave | `Leave-Application-v3.html` |
| Leave Approval | `Leave-Approval-v3.html` |
| Audit & History | `Audit-History-v3.html` |
