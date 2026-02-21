# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Interview Vibes is a single-file static HTML assessment tool that helps interviewers quantify behavioral observations during candidate interviews. It scores 18 questions across 6 categories (0-1-2 each, max 36), calculates a pass probability percentage, and generates two downloadable Markdown reports (internal PM review + candidate feedback).

## Commands

```bash
# Run Playwright E2E tests (49 tests — spins up a local server automatically)
npm test

# Run Playwright tests with browser visible
npm run test:headed

# Run Python scoring logic tests (38 tests across 9 candidate profiles)
npm run test:scoring
# or directly:
python3 test_scoring.py
```

No build step or server required. Open `index.html` directly in a browser.

## Architecture

### index.html (single-file app)

All CSS, HTML, and JavaScript are embedded in one file (~80KB). No external dependencies.

**Design system**: Dark Disney theme (#040714 background, #f9f9f9 text, #0063e5 accent, Avenir Next font, glass-morphism cards with backdrop-filter blur). Matches the style of `calculator.html`.

**Key JavaScript structures** (all inline in `<script>` at bottom of file):
- `CATEGORIES` — maps 6 category keys (comm, story, tech, own, solve, vibe) to their question arrays
- `QUESTION_TEXT` — maps all 21 question IDs to display strings
- `HELP_CONTENT` — 21 entries with `title`, `objective`, `ask[]`, `lookFor[]` for the help modal
- `SUGGESTIONS` — per-category templates for report generation (strength/concern text, tips)
- `VERDICTS` — threshold array: 83%+ Strong Pass, 67-82% Likely Pass, 50-66% Borderline, 33-49% Unlikely, 0-32% No Pass

**Key functions**:
- `calculate()` — main scoring loop, updates sidebar badges/bars in real-time
- `updateProbeIndicators()` — toggles "Optional"/"Recommended" on probe dropdowns when any tech/own question scores 0
- `showHelp(questionId)` / `closeHelp()` — help modal with interviewing tips
- `generateInternalReport()` / `generateCandidateReport()` — builds Markdown strings for download
- `downloadMarkdown(content, filename)` — Blob/URL.createObjectURL file download
- `resetAssessment()` — clears all inputs, closes probe dropdowns

**Probe questions**: 8 optional probes (tech-p1, tech-p2, own-p1, solve-p1 through solve-p5) live inside `<details>/<summary>` dropdowns. They are tracked separately and do NOT affect the main score. The probe tag changes from "Optional" to "Recommended" when any question in that category scores 0.

### test_scoring.py

Python unittest suite that replicates the JS scoring algorithm. Contains:
- `calculate_score(answers)` — Python mirror of the JS `calculate()` function
- 9 candidate profiles (STRONG_CANDIDATE, AI_ONLY_CANDIDATE, GOOD_TALKER_BAD_DOER, NERVOUS_BUT_CAPABLE, etc.)
- 6 test classes: TestScoringEngine, TestVerdicts, TestProbeRecommendations, TestCandidateProfiles, TestStrengthsAndConcerns, TestEdgeCases

**Scoring parity**: When modifying scoring logic in the HTML, the same change must be reflected in `test_scoring.py`'s `calculate_score()` function and vice versa. The two must stay in sync.

### Content files

- `part-1.md`, `part-2.md`, `part-3.md` — Interview philosophy articles (Spanish). These inform the question design but are not consumed by the app.
- `calculator.html` — Separate tool (style reference only). Do not modify unless asked.

## Scoring Rules

- 18 base questions × 2 points = 36 max
- Probe questions tracked separately, never added to main total
- Missing answers default to 1 (neutral)
- Category strength threshold: score >= 5 out of 6
- Category concern threshold: score <= 2 out of 6
- Weak category threshold: score <= 3 out of 6 (triggers action items in reports)
