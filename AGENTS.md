# AGENTS.md

Guidance for AI coding agents working in this repository.

## Project overview

`gratitude-journal` is a single-page gratitude journaling app ("感謝日誌"), written
in Japanese for a Japanese-speaking audience. It has two daily prompts:

- **朝 (morning)**: a one-tap gratitude affirmation.
- **夜 (night)**: a short form with four text fields (頑張ったこと / ありがとうと
  言われたこと / 当たり前だけどありがたいこと / 嫌な出来事からの学び).

It also tracks a daily streak, renders a monthly calendar of completed entries,
shows recent history, and lets the user export all data as CSV or JSON.

## Structure

- `index.html` — the entire application: markup, CSS, and JS in one file. There
  is no build step, no bundler, and no package.json.
- `README.md` — just the project name.

There is no separate source tree, no test suite, and no CI configuration.
Treat `index.html` as the single source of truth.

## Running / previewing

Open `index.html` directly in a browser, or serve the directory with any
static file server (e.g. `python3 -m http.server`) and visit it. No install
step is required.

## Data model

State lives entirely in `localStorage` under the key `gratitude-journal-v1`:

```js
{
  entries: {
    "YYYY-MM-DD": {
      morning: true,               // boolean, present once the morning prompt is done
      night: {                     // present once the night form is saved
        ganbatta: string,
        arigatou: string,
        atarimae: string,
        manabi: string
      }
    }
  },
  streak: number,        // recomputed on every render, not hand-edited
  lastEntryDate: string  // currently unused by logic, kept for compatibility
}
```

Dates are always local-timezone `YYYY-MM-DD` strings (see `todayStr()`). There
is no backend and no network calls — everything is client-side and per-browser.

## Conventions when editing `index.html`

- Keep it a single self-contained file unless the user explicitly asks to
  split it up. Don't introduce a build pipeline or framework for small changes.
  - Style is plain CSS scoped under `#gj-root`, with class names prefixed
    `gj-` (BEM-ish, no CSS framework).
  - Rendering is manual DOM string templating (`innerHTML` + `addEventListener`
    wiring after each render) — not a virtual DOM. Any new interactive element
    needs its listener re-attached after the containing element's `innerHTML`
    is replaced.
  - User-provided text must go through `escapeHtml`/`escapeAttr` before being
    interpolated into `innerHTML`, since journal entries are free text.
  - UI copy is in Japanese; match the existing tone (casual but warm, per the
    field labels and button text) when adding new copy.
  - CSV export prepends a UTF-8 BOM (`﻿`) and uses `csvEscape` for
    quoting — reuse it rather than hand-rolling CSV formatting.

## Testing changes

There is no automated test suite. Verify changes manually in a browser:
exercise both tabs, save a night entry, confirm the streak/calendar update,
and confirm CSV/JSON export still downloads correctly.
