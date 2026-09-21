# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A single-file, dependency-free TODO app (Japanese UI) with real-time search and inline editing. `index.html` holds all HTML, CSS and JavaScript. There is no build step, package manager, linter or test suite — open `index.html` directly in a browser to run it and reload to see changes.

## Architecture

All logic lives in one IIFE at the bottom of `index.html`. The design is a plain state-plus-full-re-render loop:

- **State**: `todos` (array of `{id, text, done}`, newest first via `unshift`) and `editingId` (which item is in inline-edit mode). Persisted as JSON in `localStorage` under the key `todo-app-items`; `load`/`save` swallow storage errors so the app still works when storage is blocked.
- **Rendering**: `render()` rebuilds the whole `<ul>` with `replaceChildren()` on every change (add, toggle, delete, edit, search input). Every mutation must call `save()` (if it changes data) and then `render()`. DOM is built with `createElement`/`textContent`/`append` — never `innerHTML` — so user text is not interpreted as HTML.
- **Search** (`norm`, `findRanges`, `appendHighlighted`): comparison is done on NFKC-normalized, lowercased text, so full-width/half-width and case differences match. Because NFKC can change string length, `findRanges` normalizes one code point at a time and keeps a `map` from normalized positions back to indices in the original text so `<mark>` highlights land on the right characters. Keep the filtering in `render()` and the highlighting in `findRanges` consistent if you change normalization.
- **Inline editing**: `startEdit`/`cancelEdit`/`commitEdit`. `commitEdit` guards on `editingId !== t.id` because Enter triggers a re-render that removes the input and fires `blur`, which would otherwise commit twice. The keydown handler ignores `e.isComposing` so an IME-confirming Enter doesn't commit the edit. Committing an empty value keeps the original text. After render, the edit input is focused and selected.
- **Search input** uses the `input` event (not `keyup`/`change`) deliberately so results update during IME composition.

## Conventions

- UI strings and code comments are in Japanese; keep new user-facing text and comments in Japanese.
- Theming is via CSS custom properties on `:root`, with a `prefers-color-scheme: dark` override — add new colors as variables in both blocks.
- Keep the app as a single self-contained `index.html` (README documents it as the only file); update `README.md`'s feature list when behavior changes.
