# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project shape

Ephemera is a single-page PWA — a meditative "breathing light" that recolors itself based on the sun's elevation at the user's location. There is **no build system, no package manager, no tests, no lint config**. The entire app is `index.html` with inline `<style>` and `<script>` blocks. `manifest.json` and `icon.svg` are the only other runtime assets.

To develop, open `index.html` in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`) so the manifest and geolocation work over a real origin. Iteration is just edit-and-reload.

## Design rules — read before changing anything user-facing

Ephemera is a contemplative resting place, not an information display. Many apparently helpful changes would betray the project. Read these rules before proposing any user-facing change.

1. **No text in the experience layer.** The app delivers no language to the user as content. No labels, no captions, no tooltips, no status text, no onboarding, no "tap to learn more." Language is the medium of knowing; Ephemera is a medium of feeling. The moment text appears, the mind shifts from resting in the field to parsing it.

2. **Real data renders into the field, never into words or numbers.** Every computed value (sun position, moon phase, seasonal time, session drift) must translate into a change in color, light, rhythm, depth, or presence. No value is ever displayed as a number or a phrase. This is the rule that keeps Ephemera from sliding back into a dashboard.

3. **The only exception: a hidden settings surface.** A long-press (~2 seconds) is the only gesture that conjures text. It opens a utilitarian preferences panel, visually distinct from the field, dismissible by tapping outside. Settings only — no tutorials, no explanatory copy. This layer does not yet exist; when built, it is the one place language belongs.

4. **Preserve the feeling, not just the code.** The breathing curves, palette transitions, gradient softness, and first-paint sequence are tuned for emotional effect. A change that is technically cleaner but feels different is a regression. When in doubt about a visual or motion change, surface the question rather than commit.

5. **Subtraction is a feature.** Removing things that no longer serve the experience is preferred over adding things that might. The app practices ephemerality on its own behalf.

6. **No urgency, no productivity register.** Nothing in this app counts, tracks, achieves, completes, or notifies. No progress bars, streaks, achievements, scoring, gamification, or completion states. The aesthetic resists the phone's native language of alerts.

## Workflow conventions

- Commit directly to the `main` branch. This project uses no PR workflow.
- Write commit messages that describe the felt or experiential change, not just the code change. "Warm the palette in the hour before sunrise" is better than "update PAL keyframe table."
- When a change touches the breathing rhythm, palette, or first-paint sequence, note in the commit message that it was a deliberate emotional decision, not just a technical refactor.

## Architecture

Two self-invoking IIFEs in the inline script — keep them isolated; they share no state.

**IIFE 1 — breathing + palette** (the bulk of the script). Three concentric absolutely-positioned `<div>`s (`.breath-core`, `.breath-glow`, `.breath-bloom`) get sized and positioned in JS every frame; their `background-image` is a `radial-gradient(...)` rebuilt by `buildGrad()` whenever the palette changes. Three concerns are layered:

- **Solar palette.** `solarPos(lat, lon)` returns sun elevation + rising/setting flag. `PAL` is a keyframe table indexed by elevation (-90° to +90°); `computePalette()` linearly interpolates between adjacent entries and applies a small dusk/civil-twilight blue-shift when `!rising`. The output drives both the page-level CSS variables (`--bg-center`/`--bg-mid`/`--bg-outer`/`--bg-edge`/`--bg-y`, declared via `@property` so they animate smoothly) and the three breath gradients. Recomputes every 60s.
- **Geolocation.** Cached in `localStorage` under `ephemera.location`. Falls back to a hardcoded lat/lon (~Chicago suburbs) if permission denied or unavailable. The cache has no TTL — wipe `localStorage` to refresh.
- **Breathing animation.** `requestAnimationFrame` loop with phases inhale → rest → exhale, each phase's duration jittered by `vary()` per cycle. Scale runs from `minScale` to `maxScale` of `vmax()` (so the light can extend beyond viewport edges); opacity is also animated. `easeInhale`/`easeExhale` are hand-tuned piecewise curves — be careful editing them, they're load-bearing for the visual feel.

The first paint deliberately bypasses CSS transitions (sets `transition:'none'`, forces reflow with `root.offsetHeight`, removes the inline rule), then enables a 3s transition for the palette fade-in, then drops to a 60s transition for ongoing minute-by-minute drift. Preserve this sequence when touching palette init.

**IIFE 2 — apple-touch-icon generator.** Renders the icon to a 180×180 canvas at runtime and assigns its data URL to `<link id="apple-icon">`. This exists because iOS home-screen icons can't use SVG. Keep it in sync with `icon.svg` if either changes.

## Conventions

- **Compact JS style.** Single-letter vars, packed expressions, semicolons on the same line. This is intentional — keep new code in the same style rather than reformatting.
- **No frameworks, no dependencies, no transpilation.** Code must run as-is in the browser. ES5-ish syntax is used throughout (`var`, function expressions); match it.
- **No external network calls.** The app works fully offline once loaded; don't add fetches, analytics, or CDN imports.
- **CSS custom properties with `@property`** are how palette transitions animate. New animatable color/position values should follow the same `@property` pattern.
