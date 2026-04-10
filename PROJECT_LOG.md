# PROJECT_LOG.md — Wortschatz-Trainer

Running log of decisions, issues, and changes. Newest entries first.

---

## 2026-04-10 — Session 3: Animal PIN auth system

**What was built:**
- Animal selection screen: grid of 25 animals, taken ones show 🔒
- PIN system: 4-character PIN per animal, stored in Firestore `students/{animal}`
- Session persistence: localStorage `ws_animal` + `ws_pin`, validated against Firestore on load
- Login flow: pick taken animal → enter PIN → validated → start screen
- Register flow: pick free animal → create PIN → saved to Firestore → start screen
- Logout: "Wechseln" button on start screen clears session
- `startScreen` now hidden by default — `init()` decides what to show after Firestore check

**Firestore collections now:**
- `results` — quiz results `{animal, series, correct, total, ts}`
- `students` — animal registrations `{pin, created}`, doc ID = animal name

**Decisions:**
- PIN stored as plaintext in Firestore — acceptable for classroom use, not real security
- No dedup mechanism beyond Firestore doc (first to register claims the animal)
- Race condition possible if two students pick same free animal simultaneously — very unlikely in practice

---

## 2026-04-10 — Session 2: Security audit & refactor

**State:** Single `index.html` — ~1246 lines. Firebase Firestore integration live. Deployed via Vercel.

**Bugs found & fixed:**
1. **XSS in mistakes list** — `r.typed` (raw user input) injected straight into `innerHTML` in `showResults()`. A student could type `<img src=x onerror=alert(1)>` and it would execute. Fixed: added `esc()` HTML-escaping helper, applied to all user/external data in `innerHTML`.
2. **XSS in teacher grid** — Firestore data (`correct`, `total`) rendered via `innerHTML`. Since Firestore rules are wide open, anyone can write arbitrary HTML. Fixed: coerce values to `Number()` before rendering.
3. **XSS in buildDotHistory** — localStorage values injected into `title` attribute without escaping. Fixed: wrapped in `esc()`.
4. **Redundant ANIMALS array** — Identical data to `Object.keys(ANIMAL_EMOJI)`. Removed, now derived.
5. **listen-btn centering bug** — Button had `display: flex` (block-level) but parent relied on `text-align: center` (only works on inline elements). Centered by accident in some browsers. Fixed: added `margin: 0 auto`.
6. **Stale CLAUDE.md and PROJECT_LOG.md** — Both said "no backend, localStorage only" but we have Firebase. Both mentioned GitHub Pages but deployment is on Vercel. Fixed: full rewrite of both.

**Known remaining issues:**
- Firestore rules are `allow read, write: if true` — fine for single-class test, must lock down before wider use
- `TEACHER_PIN` is plaintext in client JS — anyone viewing source can see it
- Animal assignment can collide (25 pool, random pick) — unlikely for one class but no deduplication
- Firebase compat SDK pinned to 10.12.2 — will go stale

---

## 2026-04-10 — Session 1: Initial audit, cleanup, Firebase integration

**State:** Single `index.html` — started at ~1018 lines, ended at ~1246.

**What was built:**
- Firebase Firestore integration (compat SDK via CDN, no build step)
- Anonymous student identity via random animal names (25 animals, stored in localStorage)
- Teacher dashboard at `?teacher` URL param, PIN-protected
- Teacher view: grid of animals × series, last score + attempt count, color-coded

**Bugs fixed:**
1. `renderSeriesGrid` hardcoded `for (let i = 1; i <= 10; i++)` — won't pick up new series. Fixed to iterate `Object.keys(SERIES)`.
2. `speakWord()` and `speakFlashWord()` duplicated, differing only by button ID. Merged into `speakCurrent(btnId, index)`.
3. `window.speechSynthesis.onvoiceschanged = () => {};` was a no-op — removed.
4. `isCorrectAnswer` for arrays: redundant `|| exact-match` branch (case-insensitive already covers it). Simplified.

---
