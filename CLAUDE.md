# Habit Flow — Project Context for Claude Code

## What it is
Habit Flow is a single-page habit tracker for people who want gentle, sustainable routines rather than streak-pressure and guilt. It pairs daily habit tracking with an AI coach that reads daily reflections and proposes small, kind next steps. Used personally by the developer (Quaid) and his wife — installed as a PWA on both phones via "Add to Home Screen."

## Who uses it
Two users on two devices, with completely separate localStorage. There's no backend, no accounts, no sync. Each device is its own instance. Future feature ideas around "spouse competition" exist but are intentionally deferred until the solo experience is fully baked.

## Design philosophy
- **Calm liquid glass aesthetic** — soft translucency, backdrop blur, rounded surfaces, low-contrast layering, subtle animated background glows. Pure CSS, no libraries.
- **Dark mode** is the home base (default). **Light** and **System** themes are available via a segmented toggle in Settings; the color layer is tokenized with CSS variables (`--fg`, `--ac`, `--bg`, etc.) so themes flip cleanly.
- **Calm-mentor tone** everywhere: supportive, non-judgmental, never hype or shame, never toxic positivity, never hustle-culture phrasing. Treat the user as a capable adult.
- **Designed to be low-friction and low-overwhelm** (clear focus, minimal choices, no nagging) — but never labeled or described that way in the UI.
- **No emojis except the streak flame (🔥).** Don't suggest adding others.

## Architecture
- **One file:** everything lives in `index.html`. Do not split into multiple files.
- **Vanilla JS only** — no frameworks, no libraries, no build step. The Anthropic SDK is not imported; the API is called via raw `fetch()` against `https://api.anthropic.com/v1/messages` with `anthropic-dangerous-direct-browser-access: true`.
- **Tailwind via CDN** for styling. Custom CSS in a `<style>` block for the glass effects, animations, and component patterns Tailwind doesn't cover cleanly.
- **localStorage** is the single source of truth for all state. No cookies, no IndexedDB, no service worker. PWA install works without offline support.
- **No service worker.** Offline mode was intentionally skipped (the coach call needs internet anyway).

## Data schema (localStorage)

Top-level keys, all prefixed `hf.`:

- **`hf.categories`** — array of groupings. Fields: `id` (`c_*`), `name`, `color` (hex), `order`.
- **`hf.habits`** — array of habit definitions. Fields: `id` (`h_*`), `name`, `categoryId`, `order`, `createdAt` (ISO 8601). The `createdAt` field is load-bearing — it's how the app keeps past days static when new habits are added (any habit with `createdAt > date` should NOT count toward that date's stats).
- **`hf.days`** — object keyed by date string (`"YYYY-MM-DD"`). Each day: `completions` (`{ habitId: bool }`), `reflection` (`{ wentWell, wasHard, wantDifferent }`), `submitted` (bool), `sealedAt` (ISO), and optionally `aiResponse` (`{ hint, compliance_read, suggestions: [...], hintDismissed?, generatedAt }`).
- **`hf.settings`** — single config object: `apiKey` (Anthropic), `sortMode` (`"category" | "order"`), `overallStreak`, `longestStreak`, `categoryStreaks` (`{ categoryId: number }`), `devMode` (bool), `dateOverride` (ISO or null), `theme` (`"light" | "dark" | "system"`, default `"dark"`).
- **`hf.drafts`** — ephemeral, keyed by date. Stores unsubmitted EOD reflection text so users don't lose drafts on refresh. Auto-pruned after 7 days. Cleared on seal.

## Critical invariants (don't break these)

1. **Past days are static.** Any function that computes stats for a past date must filter habits to those whose `createdAt <= date`. Functions to check: `computeStreaks`, `dayRatio`, the 7-day completion rate calc, the yesterday recap in `renderToday`, the Day Detail modal. Adding a new habit must never change past days' completion ratios or break category/overall streaks.

2. **`getToday()` is the only "today" source.** It returns `hf.settings.dateOverride` if set (dev mode), otherwise `todayISO()`. Every date-dependent computation must call `getToday()` — never `new Date()` directly for "what's today." `new Date()` is only used for real wall-clock timestamps (`createdAt`, `sealedAt`, `generatedAt`).

3. **Category groupings are sacrosanct.** The AI coach can add/tweak/remove/condense habits, but never modify a habit's `categoryId` and never modify categories themselves. Categories are user-managed only.

4. **HTML escape user strings.** All user-typed strings (habit names, category names, reflection answers) must go through `esc()` before being injected into HTML. This is consistent throughout the existing code — maintain it.

5. **Modal/overlay system is shared.** There's one `#hf-overlay` and `#hf-modal` element pair. Don't create parallel modal systems.

## Current major features
- Tab navigation (Today / Habits / Progress / Settings). Mobile uses a floating glass pill; desktop uses a top bar. Settings is its own view (not a modal).
- Custom habits & categories with full CRUD (add, rename inline, delete, reorder, drag-to-reorder on desktop)
- End-of-day review flow with sealing logic (no sliders — three short-answer textareas, locks the day on submit)
- AI coach: reflection-driven output of **2 insights, each with 2 application options**. Three states per suggestion: Apply / Maybe later / Not for me. "Maybe later" surfaces the suggestion again on a future day via a `surfaceOn` field.
- Calendar view in Progress with month navigation and per-day completion rings; tap a past day to view/edit it
- Dev mode with date override (forward arrow advances by one day) for rapid testing; tagged days get cleaned up on dev-mode exit
- PWA install (manifest + icons, safe-area handling, no-zoom inputs)
- Toast system (`showToast`) for transient feedback
- Reflection draft autosave (debounced)

## Do NOT add without explicit approval
- Real backend / server / database
- User accounts / authentication
- Voice recording or audio anywhere
- Push notifications or scheduled reminders
- Sliders (these were intentionally removed in favor of textareas)
- Archiving system (delete is the only "off" state for habits)
- Export / import
- Habit "favorites" or pinning
- Per-habit streak counters
- Stats dashboards beyond what's there
- Background music or sound effects
- Onboarding wizard
- Multi-color habits or custom icons per habit

## Git workflow
- `main` = production (URL: live, on both phones' home screens)
- `staging` = dev (URL: a separate Vercel preview deploy, only Quaid uses it)
- Work happens on `staging`. Push to staging → test → merge `staging` into `main` → push main to deploy to prod.
- See `WORKFLOW.md` in the repo root for exact commands.

## How to make changes
- Prefer **small, targeted edits**. Don't rewrite swaths of code while making a small change.
- **Never refactor unrelated code** while implementing a feature. If you notice something messy, mention it but leave it alone.
- **Ask before any schema change** (adding/removing/renaming `hf.*` keys, or adding/removing fields on existing records).
- **Match the existing aesthetic and tone** in any new copy or UI.
- **Verify the file parses cleanly** before declaring a build done (the codebase has been corrupted by partial edits before — be careful).
- **Preserve the comment-banner style** for major sections (e.g. `/* ════ MODALS ════ */`). Use semantic section names, not "CHUNK X" labels.
