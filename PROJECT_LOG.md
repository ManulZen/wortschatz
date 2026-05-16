# PROJECT_LOG.md — Wortschatz-Trainer

Running log of decisions, issues, and changes. Newest entries first.
For older entries (sessions 1–3), see [PROJECT_LOG_ARCHIVE.md](PROJECT_LOG_ARCHIVE.md).

---

## 2026-05-16 — Session 25: Simplification pass for locks and details

**What changed:**
- Removed legacy account lock UI and Firestore lock reads/writes from the active app
- Animal selection no longer disables animals based on stale `app/locks` data
- Teacher overview no longer shows locked-account counts
- Student detail overlay now opens with high-level diagnosis, warning signals, top mistakes, and compact evidence per series
- Detailed per-series timelines, typed mistakes, hint usage, retry usage, and open-state notes remain available behind a "Detailverlauf und Rohdaten anzeigen" disclosure
- Updated `AGENTS.md` to reflect the removed lock UI

**Important notes:**
- Old `app/locks` documents may still exist in Firestore, but the app ignores them
- The goal was deliberate deletion and simplification while preserving detailed logs when the teacher chooses to open them

---

## 2026-05-16 — Session 24: Weekly focus and narrower student path

**What changed:**
- Added a shared `app/settings.focusSeries` class setting for a teacher-selected weekly series
- Teacher series management now has a "Wochenserie" action and a dashboard focus panel showing the active focus and recording coverage
- Student start screen now primarily shows the weekly series when a focus is set, plus any locally unfinished dictation series so students can still resume work
- Mode selection now prioritizes "Weitermachen" when a draft exists and hides competing mode cards until the draft is continued or explicitly discarded
- Added an explicit "Angefangenes Diktat verwerfen" action that deletes the matching partial doc before returning to normal mode choices
- Updated `AGENTS.md` to document `app/settings` and focused weekly-series behavior

**Important notes:**
- Focus mode is a class-wide narrowing tool, not a permission boundary; the app still trusts the client because this is a class prototype with permissive Firebase rules
- Draft visibility still depends on localStorage, so unfinished work is resumable on the same device/browser/account

---

## 2026-05-16 — Session 23: Data-informed coaching and warning signals

**What changed:**
- Added teacher warning signals for patterns seen in the class data: unfinished dictations, many older starts, low full-dictation score, likely guessing, ineffective retries, and unused tips
- Teacher detail views now collapse stale partial docs for display, keeping the latest still-open partial per child/series and showing how many older starts were hidden
- Child quiz flow now makes first-level hints available after typing begins
- After a wrong first attempt, the app replays the word, shows the first hint, and labels the next action as listening and writing again
- Very short one-character guesses for longer words now trigger a listening prompt instead of being saved as answered attempts
- Updated student mode copy to encourage listening, quiet repetition, and conscious use of tips
- Updated `AGENTS.md` with the new hint/retry and stale-partial display behavior

**Important notes:**
- Stale partial collapsing is display-only; it does not delete old Firestore result docs
- The goal is to reduce guessing loops and make the teacher dashboard distinguish "needs help" from "has completed attempts"

---

## 2026-05-16 — Session 22: Clearer teacher detail overview

**What changed:**
- Renamed teacher-dashboard partial attempts from "angefangen" to "offene Zwischenstände"
- Reworked the student detail overlay into clearer sections: completed rounds, latest activities, mistake words, typed mistakes, tips used, and immediate retry usage
- Added an explanation box for teacher-facing terms: open intermediate state, tips, and "Nochmal probiert"
- Starting a new quiz while a local draft exists now deletes the matching Firestore partial doc before creating the new session
- Updated `AGENTS.md` to document that resume works from the same local browser cache and that explicit new attempts clean up the old partial

**Important notes:**
- Existing old partial docs are still preserved; the UI labels them more clearly instead of treating them as finished attempts
- Cross-device resume is not supported because the full quiz draft lives in localStorage, while Firestore partial docs store only dashboard progress

---

## 2026-05-16 — Session 21: Infinitiv-Werkstatt word list refresh

**What changed:**
- Updated default series 5-10 from the revised `Wortschatzwerkstatt_3Klasse (2).pdf` student worksheet pages
- Replaced several conjugated verb forms with infinitives and added the updated noun phrases from the worksheet
- Added `DEFAULT_SERIES_VERSION = "2026-05-16-infinitiv-werkstatt"` so existing Firestore `custom_series` docs for unchanged seeded class material are refreshed once
- The refresh writes only `words`, `contentVersion`, and `updated` with merge semantics, preserving existing result documents, recording metadata, other series fields, and manual custom word-list edits
- Documented optional `contentVersion`/`updated` fields in `AGENTS.md`

**Important notes:**
- The PDF's overview and solution pages still show the older lists; the app now follows the per-series student worksheet pages
- Series 5 contains `rennen` twice because the revised worksheet page contains it twice
- Existing teacher recordings for renamed words are preserved in Firestore/Storage but only matching word keys will be used by the student player

## 2026-05-05 — Session 20: Recording draft autosave on next recording

**What changed:**
- Teacher recording drafts now remember their series and word in `recorderState`
- Starting a recording for a different word automatically saves the current draft first
- If the autosave fails, the next recording does not start, so the teacher can retry without losing the draft
- `saveRecording()` now returns a success boolean for manual saves and autosaves
- Updated `AGENTS.md` to document autosave-on-next-recording behavior

**Important notes:**
- This only autosaves completed local drafts; while a recording is actively running, other recording buttons remain disabled as before
- Students still only hear a recording after the Storage upload and Firestore `audio` metadata save succeed

---

## 2026-05-05 — Session 19: Optional immediate retry for wrong words

**What changed:**
- Wrong quiz answers now offer an optional "Nochmal probieren" action before moving on
- The correct spelling is withheld after the first wrong attempt, then shown after a second wrong attempt
- Hints can be used before the retry attempt, making progressive hints useful inside the same question
- Quiz result payloads now optionally include `retriesUsed: [{word, attempts, solved}]`
- Mistakes still include wrong first attempts even when the word is solved on retry
- Teacher student details show retry usage in attempt chips, timeline rows, and a "Nochmal probiert" summary
- Documented `retriesUsed` in `AGENTS.md`

**Important notes:**
- Only one immediate retry is offered per word to avoid trapping children in a frustrating loop
- This feature is separate from the planned "Schwierige Wörter" spaced practice mode

---

## 2026-05-05 — Session 18: Partial quiz resume and clearer practice types

**What changed:**
- Added local quiz drafts so students can continue an unfinished dictation for the same animal and series
- Added a visible "Weitermachen" card on the mode screen when a saved draft exists
- Started but unfinished quizzes now save an overwriteable Firestore partial doc under `results/partial_<sessionId>`
- Partial docs use `mode: "quiz_partial"` and are updated after answered words, then deleted when the quiz completes
- Completed quizzes now store `practiceType: "full"` or `"retry"` so "Fehler üben" mini-rounds are no longer confused with full dictations
- Teacher dashboard details now label timeline rows as `Diktat`, `Fehlerübung`, or `angefangen`
- Teacher overview and performance analysis now use full dictations for score statistics instead of counting retry mini-rounds as normal dictations
- Documented partial quiz docs and local draft keys in `AGENTS.md`

**Important notes:**
- Old retry results without `practiceType` are inferred as Fehlerübungen when their total is smaller than the series length
- Partial saving records answered words only; text typed into the current field but not checked is not saved

---

## 2026-05-05 — Session 17: Swiss TTS fallback

**What changed:**
- Switched the SpeechSynthesis fallback locale back from `de-DE` to `de-CH`
- Updated voice selection to prefer `de-CH` voices before falling back to any German voice
- Updated `AGENTS.md` to document the Swiss TTS fallback choice

**Important notes:**
- Teacher recordings still take priority; this only affects the browser TTS fallback when no saved recording is available or playback fails
- The switch is based on device feedback that `de-CH` currently sounds better for the class devices despite the spelling target remaining school German

---

## 2026-05-05 — Session 16: Storage CORS and series encoding

**What changed:**
- Added `storage-cors.json` for the Firebase Storage bucket so browser uploads from the Vercel app can pass CORS preflight
- Fixed Firestore series seeding/editing so multi-answer words are stored as `{ alternatives: { "0": "...", "1": "..." } }` instead of nested arrays
- Added compatibility decoding so the app still works internally with runtime arrays and can read any legacy/manual array entries
- Updated `AGENTS.md` to document the Firestore-safe series word format

**Important notes:**
- The CORS file must be applied outside the app with Google Cloud tooling:
  `gsutil cors set storage-cors.json gs://wortschatz-2046c.firebasestorage.app`
- After deploy, missing default series 7 and 9 should seed successfully on the next app load

---

## 2026-05-05 — Session 15: Blaze Storage audio check

**What changed:**
- Confirmed teacher recordings should continue using Firebase Storage now that the Firebase project is on the Blaze plan
- Kept the existing Storage-backed audio data model: uploaded files in `teacher-audio/series-{num}/...` and metadata links in `custom_series/{num}.audio`
- Added clearer teacher-facing recording save errors for likely Billing/Blaze/quota, Storage rules, or bucket setup problems

**Important notes:**
- Real upload still needs a browser test after the Blaze upgrade has fully propagated in Firebase
- If saving still fails, check the exact alert details plus Firebase Storage rules and whether the 20 CHF billing budget has already stopped Storage access

---

## 2026-05-04 — Session 14: Student audio refresh

**What changed:**
- Student views now refresh the selected series from Firestore when a series is opened, when quiz/flashcard practice starts, and periodically before playback
- This prevents already-open student tabs from using stale `SERIES_AUDIO` metadata after the teacher switches a recording live
- Refresh failures fall back silently to the existing local series data and TTS fallback, so children are not blocked by a transient Firestore read failure

**Important notes:**
- If a recording still does not play after this refresh, check whether the Firestore `custom_series/{num}.audio` entry exists and whether its Storage download URL can be opened from the student device

---

## 2026-05-04 — Session 13: Recording stuck-state guard

**What changed:**
- Added an explicit recording save state so teacher audio controls are disabled while an upload is in progress
- Replaced the bare Storage `put()` await with a Firebase upload task that reports upload progress in the row status
- Added a 60-second upload timeout and a 20-second Firestore metadata timeout so the UI can recover from stalled network/Firebase calls instead of staying on `speichert ...`
- Kept the local draft recording after a failed save so the teacher can retry without recording the word again

**Important notes:**
- If the timeout appears on the teacher device, the likely causes are network, Firebase Storage rules, or blocked Storage access; the draft should remain available for retry

---

## 2026-05-04 — Session 12: Series action button upgrade

**What changed:**
- Upgraded the teacher series edit/delete/action buttons from plain text pills to clearer icon+label controls with distinct edit, delete, save, and recording variants
- Added stronger hover, active, and focus states while keeping the teacher dashboard compact
- Added a mobile layout rule so series action buttons wrap into usable full-width controls on narrow screens

---

## 2026-05-04 — Session 11: Recording save fix

**What changed:**
- Fixed teacher recording metadata writes so the `audio` map is replaced as one top-level field instead of being deep-merged by Firestore; this keeps saving/replacing/deleting recordings consistent
- Made recording upload metadata more robust by falling back to a safe audio content type when browsers provide an empty MIME type
- Added a guard against saving empty recording blobs, prompting the teacher to record again instead
- Preserved the interrupted UI work that clarifies recording actions and live/draft status

**Important notes:**
- Real microphone upload still needs browser testing on the teacher device; this environment can verify syntax and serve the page, but cannot grant microphone access
- If saving still fails with a Firebase permission message, check Firebase Storage rules for `teacher-audio/series-{num}/...`

---

## 2026-05-05 — Session 11: Teacher timeline and progressive hints

**What changed:**
- Teacher student details now include a per-series practice timeline with dated quiz and flashcard events
- Student detail summaries show when the student last practiced
- Added progressive student hints for repeated mistake words:
  - Hints unlock only after a word has been missed repeatedly by that student
  - Hint levels escalate from first-letter/length to spelling pattern to word shape
  - Hints require student input before the button appears and never reveal the full word
- Quiz results now optionally store `hintsUsed: [{word, level}]`
- Teacher details show hint usage per attempt and summarize which words needed hints
- Perfect-result celebration is reserved for 100% without hints; 100% with hints gets separate feedback
- Documented the optional `hintsUsed` result field in `AGENTS.md`

**Important notes:**
- Existing result documents remain compatible because `hintsUsed` is optional
- Hint behavior depends on each student's synced Firestore mistake history, so the first repeated-error hints appear after prior mistakes have been saved and synced

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
