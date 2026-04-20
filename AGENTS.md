# AGENTS.md — Athar's Math Games

This file is the entry point for any agent working in this repo. Read it before touching any file.

---

## What this project is

A collection of browser-based math games built for Athar, a kindergartener (age 5, school year 2025–2026). Each game targets specific math skills from his classroom curriculum (see `curriculum.md`). The player is always Athar.

---

## Design principles

These apply to every game:

1. **No speed pressure in early levels.** No countdown timers on levels 1–2. Level 3 may add gentle pacing but thinking time must always feel safe.
2. **Concrete → Pictorial → Abstract (CPA).** Levels 1–2 always show a visual (dot array, ten-frame). Level 3+ may fade visuals.
3. **Wrong answers retry with escalating hints — never session termination.** Failure is warm and recoverable.
4. **Hints are always free.** A "?" button is always on screen. Using it costs nothing.
5. **Celebrate effort, not speed.** Stars reward accuracy, not how fast answers came.
6. **Max 4–5 on-screen interactive items.** Working memory limit for age 5.
7. **Randomness adds, never undoes.** Random events introduce new challenges; they never remove correct work already done.
8. **Natural session length ~10 minutes.** Each game has a satisfying completion point within that window.

---

## Conventions for new games

- Filename: `game{N}-{short-slug}.html`
- Same single-file pattern: inline CSS + HTML + JS, no external dependencies
- Follow the design principles above
- All game assets should be placed in the `assets/` directory
