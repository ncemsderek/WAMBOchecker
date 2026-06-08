# NCEMS Apparatus Check App

**Version 2.0.0 · 2026-06-07**

Daily apparatus check app for **North Country EMS / Clark Fire 13**. Replaces Vector Solutions Check It. Single-file HTML + JSON, no build step, GitHub Pages friendly.

Live: https://ncemsderek.github.io/WAMBOchecker/

---

## What's new in v2.0

- **New home screen** — centered all-caps title with NCEMS round logo, two big photo-backed buttons (Ambulances / Other Units)
- **Inventory loaded** — 2,211 items across 5 ambulances from the Vector Solutions CSV export, items shown per compartment with Complete/Incomplete toggles
- **M12 retired** — removed entirely
- **Rescue Rig check** — new template for SAR 51, Rescue 51, Rescue 1, 3, 4, 7, 8 based on the Station 51 Daily Rescue Check form
- **Day-of-week rotation** — "Other Units" shows today's scheduled rig in a red banner at the top, others below in green for ad-hoc checks
- **EmailJS wired up** — submissions go to `d.boyce@northcountryems.org` via Gmail
- **No more Outside/Inside/Both picker** — any crew member can fill any section, submissions merge by unit + date downstream
- **Bottom status bar** — sticky "Ambulance check is X% complete" with flag count
- **Free-text initials** — no more dropdown roster

---

## Files

| File | Purpose |
|------|---------|
| `index.html` | The whole app — home screen, pickers, checks, submit, success. Single file. |
| `units.json` | All apparatus data, EmailJS config, compartment inventory. Fetched at runtime. |
| `logo.png` | Round NCEMS logo (shown on home screen) |
| `ambulance.jpg` | Background photo for "Ambulances" home button |
| `rescue51.jpg` | Background photo for "Other Units" home button |
| `sar51.jpg` | Spare photo of SAR 51 (Volcano Rescue) — not used in v2.0, here for future use |
| `README.md` | This document |

---

## Branding

```css
--g-deep: #1C3A1C   /* Header bars, headlines */
--g-dark: #254D25   /* Hover states */
--g-mid:  #2E6B2E   /* WNL buttons, Complete buttons */
--g-accent: #3A8A3A /* Accents */
--g-light: #C8E6C8  /* Soft borders */
--g-pale:  #EBF5EB  /* Section backgrounds */
--warn:    #B87E0A  /* Abnormal amber */
--bad:     #C13E3E  /* Critical red, due-today banner */
--info:    #1F5C8B  /* "Fixed it" blue */
```

---

## App flow

```
HOME
├── AMBULANCES  → list (M25A, M25B, M19, M18, M14)  → ambulance check
└── OTHER UNITS → list with today's rig in red banner → rescue rig check

AMBULANCE CHECK
  · Initials (free text)
  · Starting mileage
  · Walk-around notes + optional photo
  · Mechanical & Lights (14 items, WNL/Abnormal each)
  · Fluids (7 items, WNL/Abnormal each)
  · Fuel level (¼ · ½ · ¾ · Full)
  · Tire pressure (6 inputs, auto-flag out of range)
  · Electronics (Cell phone, iPad, Toughbook — Yes/No, Toughbook critical)
  · Cooler temp (Yes/No 4–9°C, opens reading + photo input on No)
  · Exterior compartments (collapsible, WNL/Abnormal + item list with Complete/Incomplete)
  · Interior compartments (same)
  · Submit → EmailJS → d.boyce@northcountryems.org

RESCUE RIG CHECK
  · Initials, Starting & Ending mileage
  · Exterior & Lights (10 items)
  · Fluids & Fuel (7 items)
  · Interior & Equipment (7 items — snow chains, map books, safety vests, radios, etc.)
  · Operational (3 items — run rig 15min, listen for noises, observe gauges)
  · Cleaning (3 items — wash, vacuum, wipe)
  · Comments box
  · Submit → EmailJS
```

---

## Day-of-week rescue rig rotation

| Day | Scheduled rig(s) |
|------|------------------|
| Sunday | SAR 51 |
| Monday | Rescue 51 (the white Tahoe; replaces old Rescue 2) |
| Tuesday | Rescue 7, Rescue 8 |
| Wednesday | Rescue 3, Rescue 4 |
| Thursday | Rescue 1 |
| Friday | (none) |
| Saturday | (none) |

Today's rig shows in a red banner at the top. All other rescue rigs are listed below in green so crews can run ad-hoc checks.

---

## Check pattern — WNL / Abnormal

All non-quantitative checks (mechanical, fluids, compartments, rescue rig items) use a two-state pattern:

- **WNL** = Within Normal Limits (green)
- **Abnormal** = something's off (amber)

Tapping **Abnormal** opens a slide-down with:
- Notes textarea (free text)
- "Fixed on the spot" checkbox (preserves accountability without punishing honest reporting)

Compartments also support an optional photo upload (planned for v2.1 — backend wiring needed).

---

## Item-level checks (compartments)

Each compartment card expands to show its item list pulled from the Vector Solutions CSV. Items show their quantity and a CTRL badge for controlled substances. Each item has Complete (✓) / Incomplete (✗) buttons. A "Mark All Complete" button at the top of each item list speeds up the workflow for fully-stocked compartments.

Examples:
- **Water Rescue Bag** (M25A): 3 PFDs, 2 Throw Bags, 2 Rescue Helmets
- **Med Vault** (M25A): EZ-IO equipment + Fentanyl, Midazolam, Ketamine (all CTRL-badged)
- **Trauma Bag** (M25A): 45 items including kling, IVs, tourniquets, etc.

---

## EmailJS configuration

Already wired in `units.json`:

```json
"submissionEmail": "d.boyce@northcountryems.org",
"emailjs": {
  "serviceId": "service_uqfd3sj",
  "templateId": "template_g3fp9a9",
  "publicKey": "8vmQwA8IEmZnYP4IJ"
}
```

The submit handler calls `emailjs.send(serviceId, templateId, params)` with:
- `unit_id`: e.g., "M25A"
- `check_date`: ISO date
- `check_type`: "ambulance" or "rescue"
- `initials`: crew member initials
- `message`: the full formatted summary
- `flags_count`: number of unresolved flags
- `to_email`: destination address

If the EmailJS template uses `{{message}}` in the body, the full summary lands in the email body. Make sure the template's "To Email" field references `{{to_email}}` (or is hardcoded to the address).

---

## Two-person check & merging

No more Outside/Inside/Both picker. Either crew member can fill any sections, submit when done, and supervisor gets the email tagged with unit + date + initials. If both crew members fill out parts and submit, supervisor receives two emails for the same unit/date with different initials — easy to merge mentally.

Live partner-progress sync (real-time "SJ is on Mechanical 4/10") is planned for v3.0 with a Firebase or Supabase backend.

---

## Hosting

GitHub Pages, no build step:

1. Push these files to the repo root (or `main` branch root):
   - `index.html`
   - `units.json`
   - `logo.png`
   - `ambulance.jpg`
   - `rescue51.jpg`
   - `sar51.jpg`
2. Settings → Pages → Deploy from branch → `main` → `/ (root)` → Save
3. Visit `https://ncemsderek.github.io/WAMBOchecker/`

Cache-busting is built in (`<meta>` cache headers + `?t=${Date.now()}` on the JSON fetch). If the iOS PWA still shows stale content, delete and re-add the home-screen app.

---

## Roadmap

### v2.0.0 (this drop)
- [x] New home screen with logo + photo buttons
- [x] M12 removed
- [x] Inventory loaded from CSV (2,211 items across 5 ambulances)
- [x] Compartment cards with item-level Complete/Incomplete
- [x] Rescue Rig Check template
- [x] Day-of-week rotation with red "Due Today" banner
- [x] Free-text initials
- [x] EmailJS submit to d.boyce@northcountryems.org
- [x] No Outside/Inside/Both — any section, any crew, merges downstream
- [x] Bottom progress bar with flag count

### v2.1 — backend depth
- [ ] Photo uploads survive submit (currently base64 inline in email body — will bloat large emails; switch to Cloudinary or similar)
- [ ] Quick cabinet photos per compartment (open-door snapshot for visual reference)
- [ ] Supervisor dashboard view showing merged submissions per unit/date

### v2.2 — Med Vault narc workflow
- [ ] Confirm-narcs question with dropdowns for actual quantities if not 100% confirmed (Fentanyl 100mcg increments, Midazolam 5mg, Ketamine 500mg)
- [ ] Controlled substance discrepancy flags supervisor

### v3.0 — live sync
- [ ] Move backend to Firebase Realtime DB or Supabase
- [ ] Live partner-progress indicator ("SJ is on Mechanical 4/10")
- [ ] Offline-first with sync queue for spotty garage WiFi
- [ ] Push notifications to supervisors on flagged units

### Later
- [ ] Out-of-service marking + dashboard banner
- [ ] Historical check browsing per unit
- [ ] Compartment walk-around order (physical sequence — user to dictate)
- [ ] Better Other Units button photo (user supplying)

---

## Open questions

- Walk-around physical sequence per unit (cab first? action area? by side of rig?)
- Where does long-term data live? (Email-only now; Firebase/Supabase for v3)
- Does the app need offline mode? (garage WiFi can be spotty)

---

## Chat etiquette for next session

- Forest green throughout
- Don't over-bold or over-format responses
- User uses casual, direct language — match it
- User is non-technical-leaning but pragmatic — explain code briefly, no jargon dumps
- Ask 1–3 questions at a time when narrowing requirements
- Mechanical/fluid/compartment checks always use **WNL / Abnormal** (two-state) — not Good/Low/Fixed
- The "Fixed" nuance lives as a checkbox inside the Abnormal slide-down, not a separate button
- Item-level checks are **Complete / Incomplete** (also two-state)
- M12 is dead. Never mention it. SQ51 belongs to the fire app, not this one.
- Deliver edits in a single message per session with version bumped
