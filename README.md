# NCEMS Apparatus Check App — Project Handoff

**Version 1.0.0 · First run · 2026-06-07**

Custom daily apparatus check app for **North Country EMS / Clark Fire 13** to replace Vector Solutions Check It. Built single-file (HTML + JSON) so it deploys to GitHub Pages with zero build step.

---

## 1. Files in this drop

| File | Purpose |
|------|---------|
| `index.html` | The whole app — home screen, unit picker, start screen, outside/inside steps, submit, success. Single file, no build. |
| `units.json` | Unit metadata + templates. Fetched by the HTML at runtime. Sits next to `index.html`. |
| `README.md` | This document. |

---

## 2. Branding

Forest green, white surface, clean utility-first design. Big tap targets for gloved hands.

```css
--g-deep: #1C3A1C   /* Header bars */
--g-dark: #254D25   /* Hover states */
--g-mid:  #2E6B2E   /* Primary tile background, "Yes/WNL" buttons */
--g-accent: #3A8A3A /* Borders, status dots */
--g-light: #C8E6C8  /* Soft borders */
--g-pale:  #EBF5EB  /* Section backgrounds */
--warn:    #B87E0A  /* Abnormal amber */
--bad:     #C13E3E  /* Out-of-range red, critical missing */
--info:    #1F5C8B  /* "Fixed it" blue */
```

Logo: placeholder. User to provide actual logo file.

---

## 3. Flow

```
Home
├── Ambulances → M25A, M25B, M19, M18, M14
└── Other Units → SQ51

Pick unit → Start screen
  · Date (auto)
  · Unit picker (chips)
  · Initials dropdown (crew roster)
  · Half picker: Outside / Inside / Both
  · "Your partner can do the other half on their phone — halves auto-merge by unit + date when both submit."

→ Outside step (if Outside or Both)
  · Starting mileage
  · Walk-around: notes textarea + optional damage photo
  · Pre-trip mechanical (WNL / Abnormal per item)
  · Fuel level (¼ · ½ · ¾ · Full picker)
  · Tire pressure (6 inputs, auto-flag out of range)
  · Tire tread & condition (WNL / Abnormal)
  · Outside compartments (WNL / Abnormal per compartment + notes + photo + Fixed checkbox)

→ Inside step (if Inside or Both)
  · Electronics: Cell phone, iPad, Toughbook (Yes/No; Toughbook flagged if No)
  · Cooler temperature: between 4–9°C? Yes/No → if No, opens reading input + thermometer photo
  · Inside compartments (same WNL pattern as outside)

→ Submit
  · Payload logged to console (jsonbin write is wired but disabled — set CONFIG.submitUrl)
  · Success screen with summary + flag count
```

---

## 4. Check pattern — WNL / Abnormal

All non-quantitative checks (fluids, leaks, tread, compartments) use a two-state pattern:

- **WNL** = Within Normal Limits (green)
- **Abnormal** = something's off (amber)

When **Abnormal** is tapped, a slide-down panel opens with:
- Notes textarea — free text describing what was wrong
- "Fixed on the spot" checkbox — preserves the anti-blame nuance (was low at arrival → topped off → check the box)
- (On compartments only) optional photo upload

The supervisor dashboard later treats `status: 'bad' && !fixed` as an open flag.

---

## 5. Fleet

| Unit | Type | Template | Ready |
|------|------|----------|-------|
| M25A | Medic | m25-medic | ✅ |
| M25B | Medic | m25-medic (shared with M25A) | ✅ |
| M19  | Medic | m25-medic | ⏳ |
| M18  | Medic | m25-medic | ⏳ |
| M14  | Medic | m25-medic | ⏳ |
| SQ51 | Rescue | sq51-rescue (placeholder) | ⏳ |

**M12 is gone.** Removed by user direction.

**SQ51** comes from the Station 51 Daily Rescue Check paper form. Currently a placeholder template — items list TBD.

---

## 6. M25 compartment structure

**Outside compartments** (12 total):
- Bags: Blue C-collar, Orange C-collar, Rope Bag, Trauma Bag, Water Rescue Bag
- Numbered: Compartment 1, 2, 3, 4, 5, 6, 7

**Inside compartments** (24 total):
- Bags & areas: Airway Adjunct Pocket, Airway Roll, Airway Bag (Cricothyroidotomy), Invasive and Rescue Airway Pocket, IV Tray, Lifepak 15, Medication Kit, Middle Airway Bag, Pediatric I-Gels, Portable Suction, Vacuum Splint Bag
- Named: Cab, Action Area, Bench Seat, CPR Seat, Fridge, Med Shelf, Med Vault (controlled substances)
- Lettered: Compartments A, B, C, D, E, F

**Rule going forward:** numbered compartments live outside, lettered compartments live inside.

The 460-item inventory from the Vector Solutions CSV export is **not yet populated** — `items: []` arrays are empty placeholders. First run uses compartment-level WNL/Abnormal. Item-level pass/fail can be layered on once CSV is loaded.

---

## 7. Specific behaviors

### Mileage
Free number input. Stored as string in payload.

### Fuel
4-button picker: ¼, ½, ¾, Full. "Full" turns green when selected. Anything else turns amber. Flagged at submit if not Full.

### Tire pressure
6 inputs. Auto-flag turns red below min or above max. Per-unit PSI ranges live in `units.json` → `templates.{templateId}.tirePositions[]`.

Defaults: front singles 75–80 PSI, rear duals 80–85 PSI. Override per unit if a different chassis is added.

### Electronics
Per-device Yes/No toggle. Devices with `critical: true` (currently just Toughbook) get a shield icon and turn the row red on "No." Critical + No = flag at submit.

### Cooler
Single Yes/No question: "Reading between 4°C and 9°C?" If No, slide-down opens with:
- Numeric reading input (°C)
- Photo upload (thermometer)

Always flags at submit if No.

### Controlled substances (Med Vault)
Currently structured in `units.json` under `controlledSubstanceHandling` but **not yet rendered in the UI**. The Med Vault compartment is just a WNL/Abnormal row for first run. The narc-specific UI (Fentanyl/Versed/Ketamine dropdowns by increment) is a v2 task.

---

## 8. Two-person check & sync

**First run: merge at submit (jsonbin-friendly).**

- Each crew member picks their half (Outside / Inside / Both) on their own phone.
- Each completes their half and submits.
- The payload is tagged with `unitId + date + half + crewInitials`.
- A future supervisor dashboard groups submissions sharing `unitId + date` into one combined report.

No live partner progress yet. If crews want to see "SJ is on Mechanical (4/10)" in real-time, see section 10 (v2).

---

## 9. Tech stack & hosting

**Current setup:**
- Single static HTML file fetches `./units.json` at load.
- No build step. Drop both files in a GitHub Pages repo and it just works.
- Submit logs the payload to console. Real submit endpoint is wired but disabled (`CONFIG.submitUrl = null`).

**To wire up jsonbin for writes:**

1. Upload `units.json` to jsonbin.io as a public read-only bin (this becomes `CONFIG.dataUrl`).
2. Create a separate write bin for submissions, or use jsonbin's collections API.
3. In `index.html`, set:
   ```js
   const CONFIG = {
     dataUrl: 'https://api.jsonbin.io/v3/b/YOUR_READ_BIN_ID/latest',
     submitUrl: 'https://api.jsonbin.io/v3/b/YOUR_WRITE_BIN_ID',  // or collections
     version: '1.0.0',
   };
   ```
4. jsonbin requires an `X-Master-Key` header for writes. Add to the `fetch()` in `submit()`.

**Cache busting:** The `index.html` includes `<meta http-equiv="Cache-Control">` headers and appends `?t=${Date.now()}` to the JSON fetch to dodge iOS aggressive caching.

---

## 10. Roadmap

### v1.0.0 (this drop)
- [x] Full single-file SPA with all screens
- [x] Forest green theme
- [x] WNL/Abnormal pattern with notes + Fixed checkbox
- [x] Fuel gauge picker
- [x] Tire PSI auto-flagging
- [x] Walk-around damage notes + photo
- [x] Electronics with critical flag
- [x] Cooler temp with slide-down detail
- [x] Compartment-level WNL/Abnormal
- [x] Two-person split (Outside / Inside / Both)
- [x] Submit payload logging

### v1.1 — wire up backend
- [ ] jsonbin write endpoint + master key
- [ ] Real photo upload (base64 inline now is fine but bloaty; consider Cloudinary or similar)
- [ ] Supervisor dashboard view (separate page or screen) showing merged Outside + Inside per unit/date

### v1.2 — populate items
- [ ] Load 460-item M25 inventory from CSV → populate `items: []` arrays in `units.json`
- [ ] Add per-item pass/fail UI inside each compartment card
- [ ] "Mark all WNL" button per compartment for fast workflows
- [ ] Med Vault narc-specific UI (Fentanyl/Versed/Ketamine dropdowns)

### v1.3 — SQ51
- [ ] Build SQ51 template from the rescue check sheet items
- [ ] Wire snow chains / map books / safety vests as separate yes/no items

### v2.0 — live partner sync (the original "see your partner's progress" ask)
- [ ] Move backend to Firebase Realtime DB or Supabase
- [ ] Live partner-progress indicator ("SJ is on Mechanical 4/10")
- [ ] Offline-first with sync queue for garage WiFi dropouts
- [ ] Push notifications to supervisors on flagged units

### Later
- [ ] Logo integration when user provides file
- [ ] Out-of-service marking + dashboard banner
- [ ] Historical check browsing per unit
- [ ] Compartment walk-around order (physical sequence — user to dictate)

---

## 11. Open questions

- Walk-around order for each unit (cab first? action area? by side of rig?)
- Are M19 / M18 / M14 close enough to M25 to share a template, or do they need their own variations?
- Where does completed-check data ultimately live long-term? (jsonbin for v1; real DB later?)
- How does the supervisor get notified — email, SMS, dashboard only?
- Does the app need to work offline? (garage WiFi spotty)

---

## 12. Chat etiquette for next session

- Forest green throughout
- Don't over-bold or over-format responses
- User uses casual, direct language — match it
- User is non-technical-leaning but pragmatic — explain code briefly, no jargon dumps
- Ask 1–3 questions at a time when narrowing requirements
- Mechanical/fluid checks always use **WNL / Abnormal** (two-state) — not Good/Low/Fixed (old pattern)
- The "Fixed" nuance lives as a checkbox inside the Abnormal slide-down, not a separate button
