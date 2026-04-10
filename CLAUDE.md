# CLAUDE.md — Wortschatz-Trainer

Project-level guidance for Claude Code. Read this before any session.

## Project Overview

Single-file German spelling trainer (Diktat) with Firebase-backed progress tracking.
- Entry point: `index.html` (HTML + CSS + JS in one file)
- Backend: Firebase Firestore (compat SDK via CDN) — stores quiz results per student
- Student identity: random animal name, persisted in `localStorage`
- Hosting: **Vercel** (auto-deploys from main)
- Language target: German (`de-CH` TTS)

## Architecture

```
index.html
├── <head>          Firebase compat SDK (CDN), Google Fonts
├── <style>         CSS variables + component styles
├── <body>          7 screens: start, modeSelect, quiz, flashcards, results, teacherLogin, teacherDashboard
└── <script>
    ├── Firebase    initializeApp, db = firebase.firestore()
    ├── SERIES      Word data — keyed by series number, values: string | string[]
    ├── ANIMAL_EMOJI  Animal→emoji map (identity pool, ANIMALS derived from keys)
    ├── TEACHER_PIN Hardcoded PIN for teacher view
    ├── State vars  currentSeries, currentIndex, words[], results[], flashIndex, myAnimal, selectedAnimal, takenAnimals
    ├── Helpers     getCanonical, isCorrectAnswer, displayWord, esc (HTML escaping)
    ├── TTS         speak(text, cb) — SpeechSynthesis API
    ├── Screens     showScreen(id), goHome(), selectSeries(n)
    ├── Identity    showAnimalSelect(), renderAnimalGrid(), pickAnimal(), confirmAnimal(), loginAs(), logout()
    ├── Quiz        startQuiz(wrongOnly), loadQuestion(), checkAnswer(), nextWord()
    ├── Flash       startFlashcards(), loadFlashcard(), flipCard()
    ├── Results     showResults() — saves to localStorage + Firestore
    ├── Firebase    saveResult() — writes {animal, series, correct, total, ts} to Firestore
    ├── Teacher     checkPin(), loadTeacherDashboard(), renderTeacherGrid()
    └── Utils       shuffle(), launchConfetti(), buildDotHistory()
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
- Collection: `results` — one doc per completed quiz `{animal, series, correct, total, ts}`
- Collection: `students` — one doc per animal `{pin, created}` — doc ID is the animal name
- Firestore rules: currently wide open (`allow read, write: if true`) — acceptable for single-class test, must be locked down before wider use
- Config is embedded in the `<script>` block (API key is not secret for Firebase client SDK)

## Student Identity

- On first visit: student picks an animal from a grid and creates a 4-character PIN
- Animal + PIN stored in Firestore `students/{animal}` and in localStorage
- On return: session auto-validated against Firestore
- Taken animals show 🔒 — can still be claimed with correct PIN (new device)
- "Wechseln" button on start screen lets students switch animal (logout)

## Teacher View

- URL: `?teacher` query param
- PIN-protected (default `1234`, stored as `TEACHER_PIN` constant)
- Shows grid: animals × series, last score + attempt count, color-coded
- The PIN is in the client JS — it's not real security, just a speed bump

## Key Rules

- **Do not split into multiple files** unless explicitly asked. Single-file is intentional for portability.
- **Do not add a build step** (webpack, vite, etc.) unless explicitly asked.
- **LocalStorage keys**: `ws_best_<n>`, `ws_history_<n>`, `ws_animal`, `ws_pin` — series number, animal name, or PIN.
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
| 2026-04-10 | Students could impersonate each other (no auth) | Added animal PIN system: pick animal + 4-char PIN, stored in Firestore `students` collection |

## What to Update Here

Whenever something breaks or a surprising decision is made, add a row to the table above.
Keep this file honest — stale docs are worse than no docs.
