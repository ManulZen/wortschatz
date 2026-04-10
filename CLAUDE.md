# CLAUDE.md — Wortschatz-Trainer

Project-level guidance for Claude Code. Read this before any session.

## Project Overview

Single-file German spelling trainer (Diktat). No build step, no framework, no server.
- Entry point: `index.html` (HTML + CSS + JS in one file)
- State: `localStorage` only — no backend
- Language target: German (`de-CH` TTS)

## Architecture

```
index.html
├── <style>         CSS variables + component styles
├── <body>          4 screens: start, modeSelect, quiz, flashcards, results
└── <script>
    ├── SERIES      Word data — keyed by series number, values: string | string[]
    ├── State vars  currentSeries, currentIndex, words[], results[], flashIndex
    ├── TTS         speak(text, cb) — uses SpeechSynthesis API
    ├── Screens     showScreen(id), goHome(), selectSeries(n)
    ├── Quiz        startQuiz(wrongOnly), loadQuestion(), checkAnswer(), nextWord()
    ├── Flash       startFlashcards(), loadFlashcard(), flipCard()
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

## Key Rules

- **Do not split into multiple files** unless explicitly asked. Single-file is intentional for portability.
- **Do not add a build step** (webpack, vite, etc.) unless explicitly asked.
- **Do not add a backend** unless explicitly asked.
- **LocalStorage keys**: `ws_best_<n>`, `ws_history_<n>` — series number is the key.
- **TTS language**: `de-CH` with fallback to any `de-` voice.
- **Case sensitivity**: single-word answers are exact-match (German capitalisation matters). Array answers allow case-insensitive match.

## Known Issues / History

| Date       | Issue | Resolution |
|------------|-------|------------|
| 2026-04-10 | Initial setup | Created CLAUDE.md and PROJECT_LOG.md |

## What to Update Here

Whenever something breaks or a surprising decision is made, add a row to the table above.
Keep this file honest — stale docs are worse than no docs.
