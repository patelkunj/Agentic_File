# DESIGN.md — Workbase / EMS design system

How every screen in this product must look and behave. It is derived from the HTML design set in
`Design/` (login, admin + employee dashboards, and the nine module screens). When building or
changing a screen, follow this file; when it and an old screen disagree, this file wins.

The rule of thumb: **reuse the tokens and the components below. Do not invent a new colour, a new
font size, a new radius, or a new card shape.**

---

## 1. Tokens

Every page starts with this exact block. Never hard-code a colour that exists here, and never add a
token without adding it to every page.

```css
:root{
  --navy:#0f3a5f; --navy-dark:#0a2740; --ink:#1e293b; --slate:#475569; --slate-light:#64748b;
  --line:#e2e8f0; --line-strong:#cbd5e1; --bg:#f3f5f8; --card:#ffffff; --tab-bg:#dbe7f2;
  --accent:#0ea5e9; --accent-soft:#e0f2fe; --good:#16a34a; --good-soft:#dcfce7;
  --warn:#b45309; --warn-soft:#fef3c7; --danger:#dc2626; --danger-soft:#fee2e2;
  --violet:#5b21b6; --violet-soft:#ede9fe;
}
*{box-sizing:border-box; font-family:-apple-system,BlinkMacSystemFont,"Segoe UI",Roboto,Helvetica,Arial,sans-serif;}
body{background:var(--bg); color:var(--ink); margin:0; padding:24px;}
```

### What each colour means

| Token | Use it for |
|---|---|
| `--navy` | Primary buttons, active tab, modal headers, section titles, numbered step circles |
| `--navy-dark` | Page title, card header text, all "strong" values (names, stat numbers) |
| `--ink` | Body text and input text |
| `--slate` | Labels, secondary button text, table header text |
| `--slate-light` | Meta text, hints, timestamps, placeholder-ish detail |
| `--line` / `--line-strong` | Hairlines / input + secondary-button borders, dashed boxes |
| `--bg` | Page background (never white) |
| `--card` | Card surface |
| `--tab-bg` | Card header band, card border |
| `--accent` / `--accent-soft` | Links, focus ring, selected state, info/"applied" badges, icon tiles |
| `--good` / `--good-soft` | Approved, active, mandatory, positive balance, increases |
| `--warn` / `--warn-soft` | Override needed, pending verification, restricted (RH), weekend flag |
| `--danger` / `--danger-soft` | Rejected, locked, delete, required `*`, negative balance, logout |
| `--violet` / `--violet-soft` | Role tags and "acts as holiday" chips only |

Off-white `#fafbfd` is the standard "inset" surface (context strips, panel footers, link cards,
balance boxes). `#f8fafc` is the standard hover/table-header surface. Keep both.

### Semantic colour rule

Status colour is meaning, not decoration: green = approved / active / mandatory, amber = warning /
override / restricted, red = rejected / locked / destructive, blue = informational / applied /
selected. Never re-use one of these for a different meaning on a new screen.

---

## 2. Type

One font stack (system UI, above). No web fonts, no second family.

| Role | Size | Weight | Colour | Notes |
|---|---|---|---|---|
| Page title `h1` | 18px | 700 | `--navy-dark` | UPPERCASE, `letter-spacing:.5px` |
| Card header | 14px | 700 | `--navy-dark` | Sentence case |
| Section title | 12px | 700 | `--navy` | UPPERCASE, `letter-spacing:.4px` |
| Field label | 11px | 700 | `--slate` | UPPERCASE, `letter-spacing:.3px` |
| Body / input text | 13px | 400 | `--ink` | |
| Table cell | 12.5px | 400 | `--ink` | |
| Table header | 11px | 700 | `--slate` | UPPERCASE |
| Meta / hint | 11px | 400 | `--slate-light` | |
| Badge | 11px | 700 | per status | |
| Stat value | 22px | 700 | `--navy-dark` | Dashboard tiles |

**Minimum font size is 10px, and only for badges and chips.** Do not go below 11px for anything the
user has to read as a sentence. (Some older screens use 9–10.5px body text — do not copy that.)

---

## 3. Shape, spacing, depth

```
Radius     : 5px inputs & buttons · 6px small chips/banners · 8px cards, panels, boxes
             9999px badges and pills · 50% avatars
Card        : background var(--card); border 1px solid var(--tab-bg);
              box-shadow 0 4px 12px rgba(15,23,42,.06)
Modal       : radius 10px; box-shadow 0 20px 50px rgba(0,0,0,.25)
Page padding: 24px (14px on phones)
Card padding: 12px 20px header · 20px body · 16px 20px strips and footers
Gaps        : 18px between cards · 14–18px inside grids · 10px between inline buttons
```

Only two elevations exist: the card shadow and the modal shadow. Do not add new shadows.

---

## 4. Page skeleton

Every screen after login is built in this order:

```html
<body>
  <div class="top-header"> logo + app name … user menu </div>   <!-- full-bleed bar -->
  <h1>Page Title</h1>
  <div class="nav-tabs"> … </div>                                <!-- role-based, see §6 -->
  <div class="screen-card"> … </div>                             <!-- main card -->
  <!-- optional further .panel blocks, modals last, then <script> -->
</body>
```

```css
.top-header{background:var(--card); border-bottom:1px solid var(--line);
  margin:-24px -24px 24px; padding:12px 24px; display:flex; align-items:center;
  justify-content:space-between; box-shadow:0 1px 2px rgba(15,23,42,.04); position:relative; z-index:30;}
.screen-card{background:var(--card); border:1px solid var(--tab-bg); border-radius:0 8px 8px 8px;
  box-shadow:0 4px 12px rgba(15,23,42,.06); overflow:hidden;}
.panel{ /* same as .screen-card but radius:8px on all corners, for cards not attached to the tabs */ }
```

The first card sits directly under the tab strip, so its top-left corner is square
(`border-radius:0 8px 8px 8px`). A card that is not attached to tabs uses `.panel` (all corners 8px).

---

## 5. Header (every page except login)

Left: a 34px gradient logo tile (`linear-gradient(135deg,var(--accent),#38bdf8)`, radius 9px) plus
the app name at 13px/700 `--navy-dark`. Right: a 36px avatar (`linear-gradient(135deg,var(--navy),
var(--accent))`), the user's name (12.5px/700) and role (11px `--slate-light`), and a chevron.

Clicking the block toggles a 230px dropdown containing: a head block (name + email on `#fafbfd`),
**Profile**, **Change Password**, a divider, and **Logout** in `--danger` linking to `login.html`.

Rules: the dropdown is **closed on load**; clicking outside closes it; the dropdown must sit above
page content (`z-index` on the header) but **below modals**; the user's name and role reflect who
uses that screen (employee screens show the employee, admin screens show the admin).

---

## 6. Navigation

Tabs are `<a>` elements, not divs, and always link to real files.

```css
.nav-tabs{display:flex; gap:4px;}
.nav-tab{padding:8px 16px; background:var(--line-strong); border-radius:6px 6px 0 0; font-size:13px;
  font-weight:600; color:var(--slate); border:1px solid var(--line-strong); border-bottom:none; cursor:pointer;}
.nav-tab.active{background:var(--navy); color:#fff; border-color:var(--navy);}
a.nav-tab{text-decoration:none; display:inline-block;}
```

**Admin tabs (7):** Dashboard · User & Access · Employee Management · Employment & Job History ·
Holiday Management · Leave Management · Audit & History.
**Employee tabs (3):** Dashboard · Apply for Leave · Holiday Calendar.

An employee must never see an admin-only tab. Exactly one tab carries `.active`, and it matches the
screen you are on. Where a module has two screens, add `.sub-tabs` below the tabs (also links).

---

## 7. Components

### Card header
```css
.card-header{background:var(--tab-bg); color:var(--navy-dark); font-weight:700; font-size:14px;
  padding:12px 20px; border-bottom:1px solid #bfdbfe; display:flex; justify-content:space-between;
  align-items:center; flex-wrap:wrap; gap:10px;}
.pill{font-size:11px; font-weight:600; background:#fff; color:var(--navy-dark); padding:3px 10px; border-radius:9999px;}
```
Left side: the card's name. Right side: a `.pill` with context ("4 pending", "Viewing as: Admin"), a
`.btn-sm`, or a link — never all three.

### Buttons
```css
.btn{padding:7px 14px; border-radius:5px; font-size:12px; font-weight:600; cursor:pointer; border:none;}
.btn-primary{background:var(--navy); color:#fff;}
.btn-secondary{background:#fff; border:1px solid var(--line-strong); color:var(--slate);}
.btn-danger{background:var(--danger-soft); color:var(--danger); border:1px solid #fca5a5;}
.btn-danger:hover{background:var(--danger); color:#fff;}
.btn-ghost{background:transparent; color:var(--slate-light); border:none;}
.btn-sm{padding:4px 10px; font-size:11px;}
```
One primary action per view; everything else is secondary or ghost. **Primary is always navy** (one
old screen uses `--accent` for it — do not copy). Destructive actions use `.btn-danger`. Cancel is
ghost and sits on the left of a footer; confirm actions sit on the right.

### Inputs
```css
label{font-size:11px; font-weight:700; color:var(--slate); text-transform:uppercase; letter-spacing:.3px;}
label .req{color:var(--danger);}
input[type=text], input[type=date], input[type=number], select, textarea{
  font-size:13px; padding:8px 10px; border:1px solid var(--line-strong); border-radius:5px;
  color:var(--ink); background:#fff; width:100%;}
input:focus, select:focus, textarea:focus{outline:none; border-color:var(--accent); box-shadow:0 0 0 3px var(--accent-soft);}
.field-note{font-size:11px; color:var(--slate-light);}
```
Label above field, `*` in red for required, one `.field-note` underneath for rules or auto-calculated
values. Every input needs an `id` and its label a matching `for` (older screens miss this — fix it in
new work). Never let filter-bar controls inherit `width:100%`; scope the rule or give filters their
own widths so a filter row stays one row.

### Forms
```css
.form-grid{display:grid; grid-template-columns:repeat(4,1fr); gap:16px 18px;}
.form-group{display:flex; flex-direction:column; gap:5px;}
.form-group.span-2{grid-column:span 2;} .form-group.span-4{grid-column:span 4;}
.section-title{font-size:12px; font-weight:700; color:var(--navy); text-transform:uppercase;
  letter-spacing:.4px; padding:16px 4px 10px; border-bottom:1px solid var(--line); margin-bottom:16px;}
.section-title .num{display:inline-flex; align-items:center; justify-content:center; width:18px; height:18px;
  border-radius:50%; background:var(--navy); color:#fff; font-size:10px; margin-right:8px;}
```
Long forms are split into numbered sections (1, 2, 3 …) and end with a footer: ghost Cancel on the
left, `Save as Draft` + primary save on the right. Modal forms use a 2-column grid instead of 4.

### Tables
```css
table{width:100%; border-collapse:collapse; font-size:12.5px;}
th{background:#f8fafc; color:var(--slate); font-weight:700; font-size:11px; text-transform:uppercase;
  letter-spacing:.3px; padding:10px 12px; border-bottom:2px solid var(--line-strong); text-align:left;}
td{padding:11px 12px; border-bottom:1px solid var(--line); vertical-align:middle;}
.table-wrap{padding:0 20px 20px;}
```
Tables live inside a card, never on the page background. The Actions column is right-aligned and
`white-space:nowrap`. Row count per `<tr>` must match the header — check it.

### Badges, chips, tags
```css
.badge{display:inline-flex; align-items:center; gap:4px; padding:3px 9px; border-radius:9999px;
  font-size:11px; font-weight:700; white-space:nowrap;}
```
Variants pair a `-soft` background with its solid text colour: `approved/active/mandatory` → good;
`applied/optional/current` → accent; `rejected/locked/delete/LOP` → danger; `override/pending/
restricted` → warn; `disabled/previous` → `#f1f5f9` + `--slate-light`; role tags → violet.
`.module-tag` (grey, radius 6px) labels which module a log line came from. `.scope-chip` shows
applicability. Leave types use `.badge-cl` (accent), `.badge-pl` (good), `.badge-lop` (danger).

### Avatars
Circle, white bold initials, `linear-gradient(135deg,var(--navy),var(--accent))`. Sizes: 26–28px in
table rows and activity lists, 36px in the header, 42–48px in context strips, 72px on the profile
banner. Other people may vary the gradient (violet/amber/green pairs) to stay distinguishable.

### Stat tiles
```css
.stat-row{display:grid; grid-template-columns:repeat(4,1fr); gap:1px; background:var(--line);}
.stat{background:#fff; padding:16px 20px;}
.stat-label{font-size:11px; color:var(--slate-light); font-weight:700; text-transform:uppercase; letter-spacing:.4px;}
.stat-value{font-size:22px; font-weight:700; color:var(--navy-dark); margin-top:5px;}
.stat-sub{font-size:11px; color:var(--slate-light); margin-top:3px;}
.stat-sub.up{color:var(--good); font-weight:600;} .stat-sub.warn{color:var(--warn); font-weight:600;}
```
Four tiles, hairline-separated by the 1px grid gap. Every tile has label, value and a sub-line that
adds context ("2 used this year"), not decoration.

### Banners
```css
.info-banner{background:#eff6ff; border:1px solid #dbeafe; border-radius:6px; padding:10px 14px; font-size:12px; color:#1e40af;}
.warn-banner{background:var(--warn-soft); border:1px solid #fde68a; color:var(--warn); font-size:12px; padding:10px 12px; border-radius:6px;}
.error-banner{background:var(--danger-soft); border:1px solid #fca5a5; color:var(--danger); …}
.success-banner{background:var(--good-soft); border:1px solid #86efac; color:var(--good); …}
```
Info banners explain a rule ("A User is a login account…"), start with ℹ️ and sit directly under the
card header. Warning banners start with ⚠ and sit next to the field they concern.

### Modals
```css
.modal-overlay{display:none; position:fixed; inset:0; background:rgba(15,23,42,.45);
  align-items:center; justify-content:center; z-index:50; padding:20px;}
.modal-overlay.show{display:flex;}
.modal-box{background:#fff; border-radius:10px; width:640px; max-width:96vw; max-height:92vh;
  box-shadow:0 20px 50px rgba(0,0,0,.25); overflow:hidden; display:flex; flex-direction:column;}
.modal-header{background:var(--navy); color:#fff; padding:14px 20px; font-size:14px; font-weight:700;
  display:flex; justify-content:space-between; align-items:center;}
.modal-body{padding:4px 20px 20px; overflow-y:auto;}
.modal-footer{display:flex; justify-content:space-between; align-items:center; gap:10px;
  padding:14px 20px; border-top:1px solid var(--line); background:#fafbfd;}
```
Widths: 460px confirm · 640px form · 980px (`.modal-box.wide`) calendar. Header is navy; use
`--danger` only for a destructive confirm and `--warn` for an override. Every modal closes four ways:
✕, Cancel, `Esc`, and a click on the overlay. Header/footer stay fixed; only the body scrolls.

### Link cards (quick actions)
```css
.link-card{border:1px solid var(--line); border-radius:8px; padding:16px; display:flex;
  align-items:center; gap:12px; background:#fafbfd; cursor:pointer;}
.link-card:hover{border-color:var(--accent); background:var(--accent-soft);}
.action-icon{width:34px; height:34px; border-radius:6px; background:var(--accent-soft); color:#0369a1;
  display:flex; align-items:center; justify-content:center; font-size:15px;}
```
Icon tile + title (12.5px/700) + one-line description + `→` in accent, four across.

### Other recurring pieces
`.emp-strip` — who this screen is about: avatar, name + ID, `dept · role · ● Active`, on `#fafbfd`.
`.add-new-box` — dashed 2px box that reveals an `.inline-form`; hover turns it accent.
`.status-tabs` / `.profile-tabs` / `.sub-tabs` — secondary tab rows, active tab white or
underlined navy. `.date-chip` — 40px calendar chip, month abbreviation over day number.
`.toggle-switch` — 38×21px pill, `--good` when on.

---

## 8. Layout patterns

- **Dashboards**: header → title → tabs → "My Overview" card (context strip + 4 stat tiles) → a
  `1.4fr 1fr` two-column `.dash-grid` of panels → a quick-actions panel. Admin screens repeat that
  block under an "Organization Overview" heading.
- **List screens**: card header → info banner → status tabs → filter bar → optional add box → table.
- **Form screens**: card header → info banner → context strip → numbered sections → footer.
- **Login**: no header, no tabs. A centred 368px card on the page background, lock icon, "Sign In",
  fields, remember me + forgot password, one full-width navy button.

---

## 9. Responsive (required on every screen)

Desktop is the design target, but nothing may overflow at 390px. Add this block to every page:

```css
@media (max-width:760px){
  body{padding:14px;}
  .top-header{margin:-14px -14px 16px; padding:10px 14px;}
  .user-info, .hdr-chevron{display:none;}
  .nav-tabs, .sub-tabs, .status-tabs, .profile-tabs{overflow-x:auto; flex-wrap:nowrap;}
  .nav-tab, .sub-tab, .status-tab, .profile-tab{flex-shrink:0; white-space:nowrap;}
  .stat-row{grid-template-columns:repeat(2,1fr) !important;}
  .form-grid{grid-template-columns:1fr !important;}
  .form-grid > *{grid-column:auto !important;}
  .dash-grid, .action-grid{grid-template-columns:1fr !important;}
  .filter-bar > *{width:100% !important;}
  table{display:block; overflow-x:auto;}    /* tables scroll inside their card */
  th, td{white-space:nowrap;}
  .field-note{white-space:normal !important;}
}
```

Check with a 390px-wide window: `document.documentElement.scrollWidth` must equal the viewport width.

---

## 10. Behaviour and code conventions

- Plain HTML, CSS and vanilla JS. No frameworks, no build step, no CDN, no external fonts. Each
  screen is one self-contained file. (When this becomes a real app, lift §1–§7 into one shared
  stylesheet and a shared header partial rather than copying them per page.)
- Show/hide is a `.show` class toggled with `classList`, never inline `style.display`.
- Interactive elements should be `<button>` or `<a>` so they are keyboard reachable. Older screens
  use `<div onclick>`; don't add more.
- Icons are emoji, inline. Keep them sparing and consistent: 📝 apply · 👤 profile · 📄 documents ·
  🕘 history/audit · 🔐 roles · 📋 review · 📅 holidays · 🔁 comp-off · ℹ️ info · ⚠ warning.
- Transitions are 0.15s or none. No animation beyond hover and open/close.
- Never use `localStorage` in these mock screens; hold state in JS variables.

---

## 11. Sample-data rules

The screens are shown to stakeholders, so the fake data has to hold up:

- "Today" is **16 Sep 2026**, and every date on every screen sits in a sensible relation to it.
  Everything is in **2026**.
- One holiday list for the whole product (12 holidays in 2026). Holiday Management, both dashboards
  and the calendar must show the same names and dates.
- The same leave request shows the same dates, type, day count and status on every screen it appears.
- Balances add up: used + remaining = entitlement, and the number matches the statistics table.
- Sunday is a weekend; Saturday follows the Saturday Off Calendar and is not auto-flagged.
- Salaries are **annual** and written `$60,000 / yr`.
- Leave types are **CL, PL, LOP** — never "CSL" or any other abbreviation.
- Keep placeholder people consistent: Admin User (System Administrator), Employee A (EMP-00214,
  Senior Developer, Engineering), Sarah Mehta (HR), J. Whitfield (Reporting Manager), Rita Sharma,
  Marcus Tan, Priya Nair, Tom Nguyen (locked account).
- Pick one country for addresses, phone numbers, currency and tax wording, and stick to it. The
  current set mixes US addresses with Indian tax (TDS) and Indian holidays — don't spread that.
- Never leave development notes in user-visible text ("…the actions we built earlier").

---

## 12. Checklist before a screen is done

1. Tokens block present and unmodified; no stray hex values.
2. Header present, menu closed on load, correct persona, Logout → `login.html`.
3. Correct role's tab set, exactly one `.active`, tabs are working links.
4. One primary (navy) action; destructive actions red; Cancel ghost on the left.
5. Labels `for` their inputs; required fields marked; focus ring visible.
6. Every table row has the same number of cells as its header.
7. Modals close four ways and sit above the header.
8. No horizontal scroll at 390px; tabs scroll, forms stack, tables scroll in-card.
9. Dates, names, balances and holidays agree with §11 and with the other screens.
10. No console errors; no text below 11px except badges.
