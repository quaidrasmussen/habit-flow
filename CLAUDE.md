# Habit Flow

## What it is
Habit Flow is a single-page habit tracker for people who want gentle, sustainable routines rather than streak-pressure and guilt. It pairs daily habit tracking with an AI coach that reflects on how you're actually doing and offers small, kind next steps. It's for anyone who finds typical habit apps loud, judgmental, or overwhelming.

## Design philosophy
- **Calm liquid glass aesthetic** — soft translucency, blur, rounded surfaces, low-contrast layering.
- **Dark mode** as the home base.
- **Designed to be low-friction and low-overwhelm** (clear focus, minimal choices, no nagging) — but never labeled or described that way in the UI.
- **Calm-mentor tone** everywhere: supportive, non-judgmental, never hype or shame.

## Data schema (localStorage, `hf.*` keys)
- **`hf.categories`** — array of groupings. Fields: `id` (`c_*`), `name`, `color`, `order`.
- **`hf.habits`** — array of habit definitions. Fields: `id` (`h_*`), `name`, `categoryId`, `order`, `createdAt` (ISO).
- **`hf.days`** — object keyed by date string. Each day: `completions` (`{ habitId: bool }`), `reflection`, `submitted`, `sealedAt`.
- **`hf.settings`** — single config object: `apiKey` (Anthropic), `sortMode`, `overallStreak`, `longestStreak`, `categoryStreaks`, `devMode`, `dateOverride`.

## Architecture
- **One file:** everything lives in `index.html`.
- **Vanilla JS only** — no frameworks, no libraries, no build step.
- **Tailwind via CDN** for styling.
- **localStorage** is the single source of truth for all state.
- **Anthropic API called directly from the browser**, using the key stored in `hf.settings`.

## Current features
- Tab navigation.
- Custom habits & categories with full CRUD.
- AI coach: reflection-driven output of **2 suggestions, each with 2 options**.
- Calendar view.
- Dev mode for testing.
- PWA install (manifest + icons).

## Do NOT add without my okay
- Real backend / server
- User accounts / auth
- Voice recording
- Push notifications
- Sliders (these were intentionally removed)
- Archiving system
- Export / import

## How to make changes
- Prefer **small, targeted edits**.
- **Never refactor unrelated code** while making a change.
- **Ask before any schema change** (`hf.*` shape, keys, or fields).
- Keep the calm-mentor tone and liquid-glass aesthetic consistent with what's already there.
