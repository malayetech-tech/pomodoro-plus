# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Pomodoro+ (`Pomodoro+` / 🍅) is a single-file Pomodoro timer web app, in French. The entire application — HTML, CSS, and JavaScript — lives in [index.html](index.html). There is no build step, no package manager, no dependencies, and no test suite; it's meant to be opened directly in a browser. See [README.md](README.md) for a user-facing overview.

## Commands

There is no build/lint/test tooling in this repo. To develop:

- **Run it**: open `index.html` directly in a browser, or serve the directory (e.g. `npx serve .` or `python -m http.server`) if you need a non-`file://` origin (required for the Notifications API to behave consistently across browsers).
- **Iterate**: edit `index.html` and reload the browser — there is no compilation or hot reload.

## Architecture

Everything is in one file, organized top to bottom as:

1. **`<style>`** — theming via CSS custom properties on `:root` (`--bg`, `--accent`, `--focus`/`--short`/`--long`, etc.), with a `prefers-color-scheme: dark` override block. Each timer mode also sets a `--mode-color` custom property (inline, via JS) that cascades into buttons and the progress ring so the accent color changes with the active mode.
2. **`<body>`** — static markup shell (mode switcher, SVG progress ring, stats row, task list) that JS fills in; there are no client-side templates.
3. **`<script>`** — a single IIFE (`(function () { "use strict"; ... })()`) containing all application logic. Key parts:
   - **`state`** — one plain object holding everything: current `mode` (`focus`/`short`/`long`), `remaining` seconds, `running`, `endAt` (absolute timestamp used to survive tab throttling/backgrounding), `tasks[]`, `activeTaskId`, and `stats` (`total` + per-day counts in `dates`). Loaded via `load()` and persisted via `save()` to `localStorage` under the key `"pomodoroPlusState"`.
   - **Render loop pattern** — state mutations are followed by a call to `render()` (or the narrower `renderStats()`/`renderTasks()`), which re-derives the entire DOM from `state`. There's no diffing; `renderTasks()` clears and rebuilds `#taskList` from scratch on every call.
   - **Timer mechanics** — `startTick()` sets `state.endAt = Date.now() + remaining*1000` and polls every 250ms via `tick()`, recomputing `remaining` from wall-clock time rather than decrementing a counter, so the timer stays correct across throttled/backgrounded tabs. On load, if a session was mid-flight (`state.running && state.endAt`), it's resumed or completed based on elapsed time.
   - **Session completion** (`completeSession()`) — on a completed `focus` session, increments today's count and the running total, credits the active task's pomodoro count, and auto-advances to a `long` break every 4th completed focus session that day, otherwise a `short` break. Completing a break always returns to `focus`.
   - **Feedback** — `playChime()` synthesizes a three-note chime with the Web Audio API (no audio assets), and `notify()` fires a browser Notification if permission was already granted; permission is requested lazily on the first click of the start button.

## Conventions

- No frameworks or external libraries — vanilla DOM APIs only.
- UI copy is in French; keep new copy consistent with that.
- All persistence is client-side (`localStorage`); there is no backend.
