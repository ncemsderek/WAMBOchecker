# NCEMS Apparatus Check App (WAMBOchecker)

**Version 2.11.0 · 2026-06-11**

Daily apparatus check app for **North Country EMS / Clark Fire District 13**. Replaces Vector Solutions Check It. Single-file HTML + JSON, no build step, hosted on GitHub Pages. Built field-first: big tap targets, gloved-hand friendly, forest green throughout.

- Live: [https://ncemsderek.github.io/WAMBOchecker/](https://ncemsderek.github.io/WAMBOchecker/)
- Repo: [https://github.com/ncemsderek/WAMBOchecker](https://github.com/ncemsderek/WAMBOchecker)

---

## Fleet

Ambulances: **M25A, M25B** (identical layout), **M19, M18, M14**. M12 is retired.
"Other Units" covers the rescue rig rotation by day of week. Fire units (E51/B51/T51/SQ51) are not in this app.

---

## How a check works

1. Pick a unit. Engine-brake notice at the top is named per rig (M18 notes its brake is automatic).
2. Initials (forced UPPERCASE) + mileage, with a **Jump to Interior** shortcut button.
3. Walk-around notes + photo → Mechanical & Lights → **Cab items** → Fluids → Fuel → Tires.
4. Exterior compartments 1–7, then interior areas in walk order: Med Vault → Fridge (temp check nested inside) → Action Area → Gurney → Bench Seat → Compartment A → CPR Seat → B–F → Lifepak 35 → Medication Kit → Airway Bag → Med Shelf → IV Tray → Pediatric I-Gels → Portable Suction.
5. Submit. Reports **≥90% complete send immediately**; below 90% they're held and auto-send at **5:00 PM** the same day (requires the app to be open at/after 5 PM on some device). A second submitter on the same unit + date shows both sets of initials.

### Check conventions

- Mechanical/fluid checks are **WNL / Abnormal**. Item checks are **Complete / Abnormal** (✓ / ✗). Two states only.
- Any button can be **tapped again to un-select it** (mistake recovery).
- "Fixed on the spot" lives inside the Abnormal slide-down — fixed issues don't flag.
- **Battery/operational checks are mandatory** — submit is blocked until every one is answered. A "No" opens a notes box with Swapped / Fixed on the spot.
- The main O2 level check (above 500 PSI) is nested under Ambulance Main O2 in Compartment 1.
- **Red tags**: bags with red tags remember the last-entered last-4 per unit — next check it's pre-filled with a one-tap **Confirm** button.
- M25A/M25B have sealed transmissions — shown as "Sealed — not needed," excluded from completion %.
- Tap the bottom progress bar to jump back to the last item you touched.

---

## Files

| File | Purpose |
|------|---------|
| `index.html` | The whole app. Single file. |
| `units.json` | All apparatus data, walk order, EmailJS config, unit notices. Fetched at runtime. |
| `logo.png` | NCEMS logo — home screen + home-screen app icon |
| `ambulance.jpg`, `rescue51.jpg`, `sar51.jpg` | Home button background photos |
| `README.md` | This document |

---

## Email reports

Submissions send via EmailJS to the configured address in `units.json`. Format is flags-first: a "⚠ N FLAGS — ACTION REQUIRED" block (or "✅ ALL CLEAR") at the top, full detail below. Flags: `[ABNORMAL]`, `[CRITICAL]`, `[LOW FUEL]`. Photos upload to Cloudinary and appear as clickable links.

---

## Admin mode

Tap the home-screen logo — it flips to an admin login. Admins can rename/add/remove items and quantities per unit.

**Important:** admin edits save to localStorage **on that device only**. To push fleet-wide: tap **Download units.json** in admin, then upload it to this repo. Devices with their own local admin edits will keep seeing those edits layered on top of the repo file — clear local overrides after a repo update if things look stale.

---

## Deploying changes

1. Upload `index.html` / `units.json` to the repo root: [https://github.com/ncemsderek/WAMBOchecker/upload/main](https://github.com/ncemsderek/WAMBOchecker/upload/main)
2. GitHub Pages redeploys automatically (give it a minute).
3. **iOS caches hard** — crews must delete the home-screen app and re-add it to pick up the new version. A normal reload is not enough. The current version shows in the home-screen footer.

---

## Branding

```css
--g-deep:   #1C3A1C  /* header bars, headlines */
--g-mid:    #2E6B2E  /* WNL / Complete buttons */
--g-accent: #3A8A3A  /* accents */
--g-pale:   #EBF5EB  /* section backgrounds */
--warn:     #B87E0A  /* abnormal amber */
--bad:      #C13E3E  /* critical red */
--info:     #1F5C8B  /* info blue */
```

---

## Version history

- **2.11.0** — flag-cycling: tap the flag count (or recap above Submit) to step through flagged items; status bar gap fix; "✓ to Complete" / "✓ All Done" labels; sealed badge layout fix; change-request emails include paste-ready JSON
- **2.10.0** — op-check notes render below the YesNo row; red * markers on red-tag bags and mandatory battery checks; fridge out-of-range BC warning; progress-bar tap fallback; admin notes field on change-request emails
- **2.9.0** — admin "Email changes to admin" button: diffs device edits against the live units.json and emails an apply-this change list
- **2.8.0** — 90%/5 PM deferred send + dual initials; mandatory battery checks with notes; fridge temp nested in Fridge; O2 check nested in Comp 1; Cab moved to mechanical area; full interior walk reorder; Comp 3/4/5/A/B/C item reorders; C-Collar bags merged; gurney converted to check items; sealed transmission badge (M25A/B); jump-to-interior; progress-bar jump-back; picker header cleanup
- **2.7.0** — interior area walk order applied across all five ambos
- **2.6.0** — unit-named engine brake notices; tap-again to un-select any button
- **2.5.0** — red tag memory: prefill + one-tap Confirm per unit/bag
- **2.4.x** — autosave, oxygen/gurney/red tags/controlled-meds box, LP35, home-button fix
- **2.3.x** — flip-card admin login, Cloudinary photos
- **2.2.0** — operational checks, airway bag grouping
- **2.0.0** — full inventory load, rescue rotation, EmailJS, M12 retired

---

## Known limits

- No backend: deferred 5 PM sends only fire if the app is open; admin sync is the manual download/upload bridge; partner progress doesn't sync live. (v3 plan: Firebase/Supabase.)
- EmailJS/Cloudinary are not BAA-capable — no real PHI.
- Admin password is hardcoded — adequate for a small department, not real security.
