# CLAUDE.md — Wortschatz-Trainer

Project-level guidance for Claude Code. Read this before any session.

## Project Overview

Single-file German spelling trainer (Diktat) with Firebase-backed progress tracking.
- Entry point: `index.html` (HTML + CSS + JS in one file)
- Backend: Firebase Firestore (compat SDK via CDN) — stores quiz results per student
- Student identity: animal name + hardcoded PIN, validated locally
- Hosting: **Vercel** (auto-deploys from main)
- Language target: German (`de-CH` TTS)

## Architecture

```
index.html
├── <head>          Firebase compat SDK (CDN), Google Fonts
├── <style>         CSS variables + component styles
├── <body>          8 screens: animalSelect, start, modeSelect, quiz, flashcards, results, teacherLogin, teacherDashboard
└── <script>
    ├── Firebase    initializeApp, db = firebase.firestore()
    ├── SERIES      Word data — keyed by series number, values: string | string[]
    ├── ANIMAL_EMOJI  Animal→emoji map (identity pool, ANIMALS derived from keys)
    ├── ANIMAL_PINS Hardcoded 4-digit PINs per animal (local auth, no network)
    ├── TEACHER_HASH SHA-256 hash of teacher password
    ├── BUILTIN_SERIES Set of hardcoded series numbers (distinguishes from custom)
    ├── State vars  currentSeries, currentIndex, words[], results[], flashIndex, lastWrongWords, myAnimal, selectedAnimal
    ├── Helpers     getCanonical, isCorrectAnswer, displayWord, esc (HTML escaping)
    ├── TTS         speak(text, cb) — SpeechSynthesis API
    ├── Screens     showScreen(id), goHome(), selectSeries(n)
    ├── Identity    showAnimalSelect(), renderAnimalGrid(), pickAnimal(), confirmAnimal(), loginAs(), logout()
    ├── Quiz        startQuiz(wrongOnly), loadQuestion(), checkAnswer(), nextWord()
    ├── Flash       startFlashcards(), loadFlashcard(), flipCard()
    ├── Results     showResults() — saves to Firestore, syncs back to localStorage
    ├── Firebase    saveToFirestore(data), syncFromFirestore() — Firestore is source of truth
    ├── Teacher     checkPin(), loadTeacherDashboard(), renderTeacherGrid(), deleteAnimalData()
    ├── Series Mgmt loadCustomSeries(), addCustomSeries(), deleteCustomSeries(), editCustomSeries(), renderSeriesManager()
    └── Utils       shuffle(), launchConfetti(), buildDotHistory(), esc(), sha256(), parseSeriesInput()
```

## Data Format

```js
const SERIES = {
  1: ["word1", "word2", ["alt1", "alt2"]],  // arrays = multiple accepted answers (first is spoken)
  2: [...]
};
```

Adding a new series: add a new key to `SERIES`. The UI auto-generates from `Object.keys(SERIES)`.

## Firebase

- Project: `wortschatz-2046c`
- Collection: `results` — one doc per completed quiz/flashcard `{animal, series, correct, total, ts, mistakes[], mode}`
- Collection: `custom_series` — teacher-created series `{words[], created}` — doc ID is the series number
- Firestore rules: currently wide open (`allow read, write: if true`) — acceptable for single-class test, must be locked down before wider use
- Config is embedded in the `<script>` block (API key is not secret for Firebase client SDK)

## Student Identity

- 25 animals with hardcoded 4-digit PINs in `ANIMAL_PINS` (zero network dependency)
- On first visit: student picks animal from grid, enters PIN → logged in
- Animal + PIN cached in localStorage (`ws_animal`, `ws_pin`), validated locally on return
- "Wechseln" button on start screen lets students switch animal (logout)
- Scores synced from Firestore on login via `syncFromFirestore()` — Firestore is source of truth
- localStorage (`ws_best_*`, `ws_history_*`) is a read cache, never written directly

## Teacher View

- URL: `?teacher` query param
- PIN-protected (SHA-256 hash in `TEACHER_HASH` constant)
- Shows grid: animals × series, last score + attempt count, color-coded
- PIN is hashed — not readable from source, but still client-side (not real auth)

## Key Rules

- **Do not split into multiple files** unless explicitly asked. Single-file is intentional for portability.
- **Do not add a build step** (webpack, vite, etc.) unless explicitly asked.
- **LocalStorage keys**: `ws_best_<n>`, `ws_history_<n>` (read cache from Firestore), `ws_animal`, `ws_pin` (session). Never write `ws_best_*`/`ws_history_*` directly — only via `syncFromFirestore()`.
- **TTS language**: `de-CH` with fallback to any `de-` voice.
- **Case sensitivity**: single-word answers are exact-match (German capitalisation matters). Array answers allow case-insensitive match.
- **XSS**: all user input and Firestore data MUST go through `esc()` or be set via `textContent` — never raw into `innerHTML`.

## Known Issues / History

| Date       | Issue | Resolution |
|------------|-------|------------|
| 2026-04-10 | Initial setup | Created CLAUDE.md and PROJECT_LOG.md |
| 2026-04-10 | renderSeriesGrid hardcoded 1–10 | Fixed to use Object.keys(SERIES) |
| 2026-04-10 | speakWord/speakFlashWord duplicated | Merged into speakCurrent(btnId, index) |
| 2026-04-10 | Added Firebase + teacher view | compat SDK via CDN, animal identity, teacher dashboard |
| 2026-04-10 | XSS in mistakes list (r.typed into innerHTML) | Added esc() helper, applied to all user/external data |
| 2026-04-10 | ANIMALS array redundant with ANIMAL_EMOJI keys | Derived from Object.keys(ANIMAL_EMOJI) |
| 2026-04-10 | listen-btn centered by text-align (wrong for display:flex) | Added margin: 0 auto |
| 2026-04-10 | CLAUDE.md/PROJECT_LOG.md said "no backend" | Updated to reflect Firebase + Vercel |
| 2026-04-10 | Students could impersonate each other (no auth) | Hardcoded `ANIMAL_PINS` — local validation, no Firestore dependency |
| 2026-04-10 | Teacher couldn't see which words students got wrong | `saveToFirestore` includes mistakes array, teacher grid shows frequent errors |
| 2026-04-10 | Flashcard sessions not tracked | `saveToFirestore({mode:'flashcard'})` on completion |
| 2026-04-10 | No way to reset student data | Teacher can delete all data for a student via trash icon |
| 2026-04-10 | No way to add series without editing code | Teacher series manager — add/edit/delete custom series in Firestore |
| 2026-04-10 | localStorage and Firestore scores diverged after teacher delete | Made Firestore single source of truth — `syncFromFirestore()` rebuilds localStorage |
| 2026-04-10 | Dual write (localStorage + Firestore) caused stale data | Removed direct localStorage writes from `showResults()` |

## What to Update Here

Whenever something breaks or a surprising decision is made, add a row to the table above.
Keep this file honest — stale docs are worse than no docs.
