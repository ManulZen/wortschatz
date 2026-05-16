# AGENTS.md — Wortschatz-Trainer

Project guidance for any coding agent working in this repo. Read this file and the newest `PROJECT_LOG.md` entry before changing code.

## Project Overview

Single-file German spelling trainer for primary school dictation practice.

- Entry point: `index.html` (HTML + CSS + JS in one file)
- Backend: Firebase Firestore and Firebase Storage via compat SDKs from CDN
- Hosting: Vercel, auto-deploys from `main`
- Student identity: animal name + hardcoded 4-digit PIN, validated locally
- Student UI goal: primary-school friendly, personal, colorful, animal-led, but still calm enough for repeated practice
- Language target: German school dictation with Swiss German TTS fallback (`de-CH`) because it sounds better on the teacher/student devices

## Mandatory Working Rule

Before implementing any user request in this repo:

1. Read `AGENTS.md`.
2. Read the newest relevant entries in `PROJECT_LOG.md`.
3. Check `git status --short --branch`.
4. Preserve unrelated user changes.
5. Update `PROJECT_LOG.md` when behavior, data format, architecture, deployment assumptions, or notable UI direction changes.

If an agent cannot follow this because of missing context or tool limits, say so explicitly before changing files.

## Architecture

```
index.html
├── <head>       Firebase compat SDKs, Firebase Storage SDK, Google Fonts
├── <style>      CSS variables and all component styles
├── <body>       student screens + teacher dashboard screens
└── <script>
    ├── Firebase       initializeApp, db = firebase.firestore(), storage = firebase.storage()
    ├── Series         DEFAULT_SERIES seed, SERIES runtime map, SERIES_AUDIO recording metadata
    ├── Identity       animal emoji/PIN maps, local login cache
    ├── Helpers        word normalization, mistake normalization, escaping, audio keys
    ├── TTS/Audio      teacher recording playback first, de-CH SpeechSynthesis fallback
    ├── Student Flow   animal select, start, mode select, quiz, flashcards, results
    ├── Teacher View   PIN login, dashboard, analytics, detail overlay, series manager
    ├── Recordings     per-word recording, local draft review, Storage upload on Save only
    └── Print Tools    teacher-only QR/login card print template builder
```

## Data Model

All series live in Firestore collection `custom_series`. `DEFAULT_SERIES` only seeds missing docs and acts as offline fallback.

```js
// custom_series/{seriesNumber}
{
  words: [
    "word1",
    "word2",
    { alternatives: { "0": "accepted1", "1": "accepted2" } }
  ],
  created: 1710000000000,
  contentVersion: "2026-05-16-infinitiv-werkstatt", // optional, for seeded default-series updates
  updated: 1710000000000, // optional, when default content was refreshed
  audio: {
    "<audioKey>": {
      word: "canonical word",
      url: "https://...",
      path: "teacher-audio/series-1/...",
      contentType: "audio/webm",
      updated: 1710000000000
    }
  }
}
```

At runtime, alternative spellings are decoded back to arrays like
`["accepted1", "accepted2"]`. Do not write nested arrays directly to Firestore;
Firestore rejects arrays inside arrays.

Quiz/flashcard results live in Firestore collection `results`.

```js
// results/{autoId}
{
  animal: "Fuchs",
  series: 1,
  mode: "quiz",          // or "flashcard"
  practiceType: "full",  // "full" or "retry" for Fehler üben
  completed: true,
  correct: 8,
  total: 10,
  ts: 1710000000000,
  hintsUsed: [
    { word: "wieder", level: 2 }
  ],
  retriesUsed: [
    { word: "nehmen", attempts: 2, solved: true }
  ],
  mistakes: [
    { word: "gross", typed: "gros" },
    { word: "wieder", typed: null }
  ]
}
```

`mistakes` must use the object format above. Legacy string mistakes are migrated from the teacher dashboard to `{word, typed: null}`.
`hintsUsed` is optional and records student-visible progressive hints used during a quiz. `level` is 1–3, where higher levels reveal more structure but never the full word.
`retriesUsed` is optional and records words that were tried again inside the same quiz question. Wrong first attempts still remain in `mistakes` even if the second attempt is correct.
Students can request a first-level hint after starting to type. After a wrong first attempt, the app automatically replays the word, shows the first hint, and steers the child toward the immediate retry. Very short one-character guesses for longer words are treated as listening prompts and are not saved as answered attempts.

Started but unfinished dictations are stored as overwriteable partial docs, not as final scores:

```js
// results/partial_<sessionId>
{
  animal: "Fuchs",
  series: 1,
  mode: "quiz_partial",
  practiceType: "full",  // or "retry"
  partial: true,
  completed: false,
  correct: 3,
  answered: 4,
  total: 10,
  started: 1710000000000,
  updated: 1710000300000,
  ts: 1710000300000,
  sessionId: "1710000000000_ab12cd3",
  mistakes: [],
  hintsUsed: []
}
```

Partial docs are updated after answered words and deleted when the quiz completes. Student resume state is also cached locally in `ws_quiz_draft_<animal>_<series>`.
Resume depends on that local browser cache, so it works on the same device/browser/account. If a child explicitly starts a new attempt instead of continuing an existing draft, the matching Firestore partial doc is deleted to avoid stale open attempts in the teacher dashboard.
The teacher dashboard collapses stale partial docs for display: per child and series it keeps the latest still-open partial, hides older partials, and hides partials older than a completed quiz for that series. This is display-only and does not delete historical Firestore docs.

## Firebase

- Project: `wortschatz-2046c`
- Firestore:
  - `custom_series` — source of truth for all series and recording metadata
  - `results` — one doc per completed quiz/flashcard
  - `app/settings` — shared class settings such as the current focused weekly series
- Storage:
  - `teacher-audio/series-{num}/...` — uploaded teacher recordings
- Current rules are permissive for the class prototype. Lock down before wider use.
- Firebase client config is embedded in `index.html`; the client API key is not a secret.

## Student Identity

- 25 animal accounts with hardcoded 4-digit PINs in `ANIMAL_PINS`
- Login cache in localStorage: `ws_animal`, `ws_pin`
- Score/history localStorage keys are read caches rebuilt from Firestore: `ws_best_<n>`, `ws_history_<n>`
- Do not write score/history localStorage directly. Use Firestore writes and `syncFromFirestore()`.
- Student-side PIN lockout and legacy lock UI are removed to prevent children from being blocked by stale lock state.
- If `app/settings.focusSeries` is set, the student start screen primarily shows that weekly series plus any locally unfinished dictation series so students have a narrower path.

## Audio And TTS

- Student playback uses teacher recording first: `getAudioMeta(currentSeries, entry)` → `playRecordedAudio()`
- If recording is absent or fails, fallback is `SpeechSynthesisUtterance` with `lang = "de-CH"` and German voice preference.
- Teacher recordings are drafts until `Speichern`:
  - Draft blob stays only in `recorderState.blob`
  - `Prüfen` plays the local draft only for the teacher
  - Upload to Storage and Firestore metadata update happen in `saveRecording()`
  - Starting a recording for a different word automatically saves the current draft first; if that save fails, the new recording does not start
  - Students see recordings only after the Firestore `audio` map is updated

## Teacher View

- Open with `?teacher`
- PIN protected by SHA-256 hash in `TEACHER_HASH` (client-side protection, not real auth)
- Shows overview cards, analysis, animal × series table, detail overlay, series management, teacher recordings, and print templates.
- Clickable student names should remain visually obvious and keyboard-accessible.

## UI Direction

The student side is for primary-school children:

- Make the child’s own animal identity prominent.
- Student account animals are exclusive to children; do not reuse animal emojis as series/session icons.
- Use non-animal symbols and colors for series/learning islands. New series automatically cycle through `SERIES_SYMBOLS` and `SERIES_COLORS`.
- Prefer animal-led identity plus color-led series orientation over generic dashboards.
- Use clear, large touch targets and obvious click states.
- Keep teacher/admin surfaces calmer and denser than student surfaces.
- Avoid merely adding gradients as “kid design”; use animals, progress, color grouping, and personal context.
- Preserve readability and avoid clutter, especially on mobile.

## Editing Rules

- Keep the app single-file unless the user explicitly asks for a larger restructure.
- Do not add a build step unless explicitly asked.
- Escape all user/Firestore content with `esc()` or use `textContent`.
- Preserve existing Firebase data shapes unless intentionally migrating them.
- When changing data format, add compatibility/migration code and document it in `PROJECT_LOG.md`.
- After changes, run at least:
  - inline JS syntax check: `perl -0ne 'print $1 if /<script>(.*)<\\/script>/s' index.html | node --check -`
  - `git diff --check`
  - HTTP smoke test when HTML/CSS changed, if feasible

## Known Gaps

- No automated browser test suite.
- Browser/microphone recording must be tested on a real supported browser, ideally the teacher’s MacBook.
- Firebase rules are not production-safe yet.
- Student UI needs real feedback from children/teacher; treat the current playful direction as iterative, not final.
