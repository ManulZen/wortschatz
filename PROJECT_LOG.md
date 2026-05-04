# PROJECT_LOG.md — Wortschatz-Trainer

Running log of decisions, issues, and changes. Newest entries first.
For older entries (sessions 1–3), see [PROJECT_LOG_ARCHIVE.md](PROJECT_LOG_ARCHIVE.md).

---

## 2026-05-04 — Session 10: Audio, student UI, and agent docs

**What changed:**
- Added `AGENTS.md` as the agent-neutral project guide; `CLAUDE.md` now points to it for compatibility
- Added a mandatory repo workflow: read `AGENTS.md`, read latest `PROJECT_LOG.md`, check git status, preserve unrelated changes, and update the log for notable behavior/data/architecture/UI changes
- Made the child-facing UI more primary-school oriented: personal animal greeting, animal parade, colored animal selection, and colored series "learning island" cards with non-animal symbols
- Corrected the student/series identity split: child accounts keep animals exclusively; series now use non-animal symbols and color palettes that cycle automatically for new series
- Improved click affordances throughout the student UI and teacher dashboard
- Teacher dashboard student names now show a clear "Details" pill, hover indication, focus state, and keyboard activation
- Recording panel now has a clear "Zuklappen" button
- Added optional teacher recordings per word:
  - Records with browser `MediaRecorder`
  - Stores files in Firebase Storage under `teacher-audio/series-{num}/...`
  - Stores recording metadata in `custom_series/{num}.audio`
  - Draft recordings remain local and are not live until `Speichern`
  - Student playback tries teacher recording first and falls back to TTS if absent or broken
- Switched fallback TTS from Swiss German locale (`de-CH`) to Hochdeutsch (`de-DE`) because Swiss primary schools teach standard German spelling/dictation
- Normalized quiz mistakes to a single object format: `mistakes: [{word, typed}]`
- Legacy string mistakes are migrated from the teacher dashboard to `{word, typed: null}` and old `mistakeDetails` is removed

**Important notes:**
- Real recording must be tested on the teacher's MacBook/browser because this environment has no browser/microphone
- Firebase Storage rules may need adjustment if saving recordings returns permission errors
- The playful UI direction is intentionally iterative; avoid future "just add gradients" changes and use animals, color grouping, child identity, and clear progress instead

---

## 2026-05-03 — Session 9: Class handoff prep

**What changed:**
- Added printable handoff file `freundin_pins.txt` with student URL, teacher URL, and all animal PINs
- Generated QR codes for the student and teacher landing pages: `qr-schueler.png`, `qr-lehrerin.png`
- Added printable browser sheet `druckvorlagen.html` with class QR sheet, teacher access, and per-student login cards
- Moved printable login cards behind the teacher login; public `/druckvorlagen.html` now redirects to the teacher view
- Fixed print CSS for the teacher-only handouts to avoid blank interleaved pages and clarified the PDF-save button
- Replaced the reused teacher password with a unique app-only password hash
- Teacher dashboard now has quick links plus overview cards for active children, strong dictation attempts, locked accounts, and most common mistake
- Teacher dashboard now includes a simple performance analysis for students needing support, difficult series, and frequent mistake words
- Teacher grid now shows all 25 animal accounts, including children with no data yet
- Teacher grid now stays usable beyond 10 series with horizontal scrolling and a sticky student column
- Teacher dashboard width and table spacing tuned so 10 series plus delete actions fit better on desktop
- Learning flow now restores mistake words from Firestore after login/reload
- Normal dictation prioritizes earlier mistake words first, then continues with the remaining words
- Student quiz now gives an empty-answer listening prompt, accepts obvious first-letter capitalization fixes, and fills the progress bar after each checked answer
- Start screen now recommends a dynamic "Heute üben" series based on missing attempts, weak scores, recent results, and known mistake words
- Disabled student-side PIN lockout to prevent children from locking each other out; teacher dashboard can still clear legacy locks
- Refreshed the child-facing look with a teal/yellow palette and stronger animal presence in start, quiz, flashcard, and result screens

---

## 2026-04-13 — Session 8: Account lockout

**What changed:**
- Students get locked out after 3 wrong PIN attempts
- Locked animals show 🔒 icon, red border, disabled — can't click or enter PIN
- Failed attempt counter stored in `localStorage` (`ws_fails_<animal>`), lock persisted in Firestore `app/locks` doc
- Teacher dashboard shows "Gesperrte Konten" bar with one-click unlock buttons
- Teacher unlock clears both Firestore lock and localStorage fail counter
- `loadLocks()` called on app init (student side) and on teacher dashboard load (refreshes state)
- `esc()` now also escapes single quotes (`&#39;`) for defense-in-depth

---

## 2026-04-10 — Session 7: Unified series in Firestore

**What changed:**
- All series (including original 10) now live in Firestore `custom_series` collection
- `DEFAULT_SERIES` in code is seed data only — written to Firestore on first run or when missing
- `BUILTIN_SERIES` removed — no more special cases, all series are equal
- Teacher can edit/delete any series (not just custom ones)
- `loadSeries()` auto-migrates: on load, seeds any `DEFAULT_SERIES` entries missing from Firestore
- Migration is safe: if Firestore write fails, already-loaded series stay in memory
- `editSeries` uses `{merge: true}` to preserve `created` timestamp

---

## 2026-04-10 — Session 6: Architecture cleanup

**Problems found and fixed:**
1. **Dual source of truth for scores** — `showResults()` wrote to both localStorage and Firestore. If Firestore write failed, next `syncFromFirestore()` wiped localStorage. Fix: removed all localStorage writes from `showResults()`. Firestore is now the single source of truth. `syncFromFirestore()` rebuilds localStorage as a read cache.
2. **Dead `students` collection** — Firestore auth was replaced with hardcoded `ANIMAL_PINS` in session 4, but docs still referenced `students` collection. Cleaned from CLAUDE.md and PROJECT_LOG.md.
3. **`loadCustomSeries` used `d.num` field** — redundant with doc ID, could diverge. Now uses `Number(doc.id)`. Removed `num` field from `addCustomSeries` writes.
4. **`teacherData` was module-level** — only used in teacher dashboard render. Now a local in `loadTeacherDashboard`, passed to `renderTeacherGrid`.
5. **`selectedAnimal` leaked after login** — never cleared. Now reset in `loginAs()`.
6. **Stale CLAUDE.md** — architecture section, student identity section, firebase section, history table all had outdated references (`takenAnimals`, `students` collection, `saveResult`, `TEACHER_PIN`). Full rewrite.
7. **Series edit missing** — teacher could add/delete custom series but not edit. Added `editCustomSeries()` with inline editing UI.
8. **Teacher password input too narrow** — 160px for a 14-char password. Widened to 260px, placeholder changed from `••••` to `Passwort`.

**Full innerHTML/XSS audit:** All 25 innerHTML assignments verified safe — every Firestore/user-sourced value goes through `esc()` or `Number()` or is set via `.textContent`/`.title`.

---

## 2026-04-10 — Session 5: Verification & refactor pass

**What was done:**
- Removed dead `teacherDocs` variable (populated but never read)
- Fixed XSS in `renderSeriesGrid` — custom series preview now wrapped in `esc()`
- Fixed XSS in `checkAnswer` — correction display now wrapped in `esc()`
- Replaced fragile `isCustom = num > 10` with `BUILTIN_SERIES` Set
- Fixed `renderSeriesGrid` key order — guaranteed numeric sort
- Merged `saveResult()` + `saveFlashcardComplete()` into single `saveToFirestore(data)`

---

## 2026-04-10 — Session 4: Features + hashed teacher PIN

**What was built:**
- Per-word mistake tracking: quiz results include `mistakes` array, teacher grid shows frequent errors
- Flashcard completion tracking: `saveToFirestore({mode:'flashcard'})` on completion
- Student data reset: teacher can delete all results for a student via trash icon
- Custom series management: add/edit/delete from teacher dashboard, stored in Firestore `custom_series`
- Hashed teacher PIN: `TEACHER_HASH` (SHA-256 via Web Crypto API)
- Hardcoded `ANIMAL_PINS` replacing Firestore-based auth (zero network dependency at login)

**Firestore collections:**
- `results` — quiz/flashcard results `{animal, series, correct, total, mistakes, mode, ts}`
- `custom_series` — teacher-created series `{words[], created}`, doc ID = series number

---
