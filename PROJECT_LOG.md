# PROJECT_LOG.md — Wortschatz-Trainer

Running log of decisions, issues, and changes. Newest entries first.

---

## 2026-04-10 — Initial audit & cleanup

**State:** Single `index.html` — 1018 lines, fully functional.

**What works:**
- 10 series × 10 words, Diktat + Flashcard modes
- LocalStorage persistence (best score, last 5 attempt dots)
- "Retry wrong words" mode
- TTS with `de-CH` locale, 0.85 rate
- Confetti on 100% score

**Bugs/smells fixed this session:**
1. `renderSeriesGrid` hardcoded `for (let i = 1; i <= 10; i++)` — won't pick up new series. Fixed to iterate `Object.keys(SERIES)`.
2. `speakWord()` and `speakFlashWord()` were duplicated functions differing only by button ID. Merged into `speakCurrent(btnId, index)`.
3. `window.speechSynthesis.onvoiceschanged = () => {};` was a no-op at the bottom — removed.
4. `isCorrectAnswer` for arrays: checked `case-insensitive || exact`, where exact is a subset of case-insensitive. Simplified to just case-insensitive for arrays.

**Not changed (intentional):**
- Single-file structure — portability is a feature
- No build tooling
- No backend
- German capitalisation enforced for single-answer words (nouns must be capitalised)

---
