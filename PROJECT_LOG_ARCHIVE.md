# PROJECT_LOG_ARCHIVE.md — Wortschatz-Trainer

Archived log entries. For current entries see PROJECT_LOG.md.

---

## 2026-04-10 — Session 3: Animal PIN auth system

**What was built:**
- Animal selection screen: grid of 25 animals
- PIN system: 4-character PIN per animal, initially stored in Firestore `students/{animal}`
- Session persistence: localStorage `ws_animal` + `ws_pin`
- Login/logout flow

**Note:** Firestore-based PIN auth was later replaced with hardcoded `ANIMAL_PINS` (session 4) because Firestore dependency at login caused failures. The `students` collection is no longer used.

---

## 2026-04-10 — Session 2: Security audit & refactor

**State:** Single `index.html` — ~1246 lines. Firebase Firestore integration live. Deployed via Vercel.

**Bugs found & fixed:**
1. **XSS in mistakes list** — `r.typed` into `innerHTML`. Fixed: added `esc()` helper.
2. **XSS in teacher grid** — Firestore data via `innerHTML`. Fixed: `Number()` coercion.
3. **XSS in buildDotHistory** — localStorage values in `title` attribute. Fixed: `esc()`.
4. **Redundant ANIMALS array** — Removed, derived from `Object.keys(ANIMAL_EMOJI)`.
5. **listen-btn centering** — Fixed with `margin: 0 auto`.
6. **Stale docs** — Rewrote CLAUDE.md and PROJECT_LOG.md.

---

## 2026-04-10 — Session 1: Initial audit, cleanup, Firebase integration

**State:** Single `index.html` — started at ~1018 lines, ended at ~1246.

**What was built:**
- Firebase Firestore integration (compat SDK via CDN, no build step)
- Anonymous student identity via random animal names
- Teacher dashboard at `?teacher` URL param, PIN-protected

**Bugs fixed:**
1. `renderSeriesGrid` hardcoded 1–10 loop. Fixed to `Object.keys(SERIES)`.
2. `speakWord`/`speakFlashWord` duplicated. Merged into `speakCurrent(btnId, index)`.
3. `window.speechSynthesis.onvoiceschanged = () => {};` no-op. Removed.
4. `isCorrectAnswer` redundant branch. Simplified.
