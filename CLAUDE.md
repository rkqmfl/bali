# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-page, mobile-first Korean-language travel companion app for a Bali / Singapore trip. Pure static HTML — no build system, no package manager, no tests, no dependencies installed at the repo level. All CSS and JavaScript are inlined in the HTML file.

## Files

- `index.html` — the app (deployed entry point).
- `bali-travel-app.html` — a byte-identical duplicate of `index.html`. The two files must stay in sync; when editing the app, apply the same change to both (e.g. `cp index.html bali-travel-app.html` after editing).
- `README.md` — placeholder, no useful content.

## Running / previewing

There is nothing to build. Open `index.html` directly in a browser, or serve the folder with any static server (e.g. `python3 -m http.server`). The viewport is locked to mobile width (`max-width: 480px`); use a phone-sized viewport or device emulation when previewing.

## Architecture

The app is one HTML document with four tabbed panels switched client-side. The structure inside `<body>` is:

```
header → nav.tabs → section.panel#exchange
                  → section.panel#checklist
                  → section.panel#itinerary
                  → section.panel#tips
```

Tab switching is wired by a single delegated handler (`index.html` ~line 1247): each `.tab` carries `data-panel="<id>"`, click toggles the `.active` class on tabs and panels.

### Currency converter (`#exchange`)

`fetchRate()` (~line 1348) tries three currency APIs in order with a 5s `AbortController` timeout each, falling through silently on failure:

1. `cdn.jsdelivr.net/npm/@fawazahmed0/currency-api`
2. `latest.currency-api.pages.dev`
3. `open.er-api.com/v6/latest/KRW`

The successful rate is cached under the `bali_rate_cache_v1` localStorage key. On load, the cached rate is shown immediately (`loadCachedRate`) and a fresh fetch starts in parallel. The default fallback rate `RATE = 11.7553` is hard-coded — update it if it drifts far from reality. KRW/IDR inputs auto-format with thousands separators on every keystroke (`handleInput`), preserving cursor position.

### Checklist (`#checklist`)

Each section's items live inside a container marked `data-list="<section>"` and have a sibling progress badge marked `data-section="<section>"`. Persistence keys items by **positional index**: `${section}_${idx}`, stored as a flat object under the `bali_checklist_v1` localStorage key.

Consequence: **reordering or inserting items in the middle of an existing list will shift everyone's saved state.** Append new items at the end of a section, or bump the storage key version (`bali_checklist_v2`) if a structural change is needed.

The progress badge initial text (`0/8`, `0/10`, …) hard-codes the section length and is overwritten by `updateProgress()` on load — keep it consistent with actual item count when adding/removing items.

### Itinerary (`#itinerary`) and Tips (`#tips`)

Static content only. Day cards (`.day-card`) and tip cards (`.tip-card`) are pure markup, no JS state.

### Storage helper

A small `storage` shim (~line 1266) wraps localStorage with an in-memory fallback (`memoryStore`) so the app keeps working in sandboxed preview environments where localStorage throws. Always use `storage.get(key)` / `storage.set(key, value)` rather than touching `localStorage` directly. `storage.get` returns `{ value }` or `null` (not the raw string).

### Silent-error policy

Global `error` and `unhandledrejection` listeners (~line 1290) suppress messages containing `"Load failed"` or `"Storage"` to keep the Claude preview console clean. Network failures inside `fetchRate` are also caught and ignored on purpose. Don't add `console.log`/`console.error` for normal flow — failures should fall through to the next source or to cached state.

## Styling conventions

- CSS custom properties in `:root` (~line 13) define the palette: `--sand`, `--terracotta`, `--jungle`, `--sunset`, `--ocean`, `--ink`, `--paper`, plus `*-deep` variants. Use these tokens rather than raw hex when adding styles.
- Fonts: `Fraunces` (serif, headings/body) and `DM Mono` (labels, metadata) loaded from Google Fonts.
- A fixed SVG noise overlay on `body::before` provides paper-grain texture; keep new full-bleed elements at `z-index: 2` (the `.container` layer) so the grain stays behind them.
- The viewport is locked (`user-scalable=no`, `maximum-scale=1.0`); design for a single mobile width and don't add desktop breakpoints unless asked.

## Language

UI strings, comments, and copy are Korean (`<html lang="ko">`). Preserve Korean text verbatim when editing — don't translate or reflow it. Code identifiers are English.
