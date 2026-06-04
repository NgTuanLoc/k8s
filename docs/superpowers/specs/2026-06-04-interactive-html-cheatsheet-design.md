# Interactive HTML Cheatsheet — Design

**Date:** 2026-06-04
**Status:** Approved

## Purpose

Turn the existing `cheatsheet.md` (Kubernetes master cheatsheet, Modules 0–14) into a
single, self-contained, interactive HTML file that's pleasant to use as a live reference
during and after the course.

## Goals

- One file: `cheatsheet.html` at the repo root.
- Zero external dependencies — inline `<style>` and inline vanilla JS. Opens by
  double-click, works fully offline (no CDN, no build step, no network).
- Three interactive behaviors: live search/filter, collapsible modules, copy-to-clipboard.
- Auto dark/light theming with a manual toggle.

## Non-Goals (YAGNI)

- No progress checkboxes / localStorage progress tracking (explicitly dropped).
- No auto-regeneration pipeline from `cheatsheet.md` (possible future follow-up).
- No build tooling, bundlers, or frameworks.
- No multi-file output / no external CSS or JS assets.

## Source of Truth

`cheatsheet.md` remains the master document. `cheatsheet.html` is a hand-built
interactive snapshot of its current content (commands + the one-line concept summary
under each module). The two are kept in sync manually.

## Content

All 15 sections currently in `cheatsheet.md`, in order:

| Module | Title |
|--------|-------|
| 0 | Contexts & clusters + Pods intro |
| 1 | Mental model: spec vs status |
| 2 | Pods: logs & exec |
| 3 | Deployments: scale, rollout, rollback |
| 4 | Deploy strategies |
| 5 | Services & DNS |
| 6 | Config: ConfigMaps & Secrets |
| 7 | Storage: PVC/PV, StatefulSets |
| 8 | Jobs & CronJobs |
| 9 | Scaling: requests/limits & HPA |
| 10 | Debugging loop |
| 11 | Health probes |
| 12 | Security & RBAC |
| 13 | Observability |
| 14 | Capstone: multi-tier with Kustomize |

Each module renders as: title → concept summary line → one or more command blocks.
Command text is copied **verbatim** from `cheatsheet.md` (including the inline `#` comments).

## Layout

- **Sticky header** containing:
  - Title ("📋 Kubernetes Master Cheatsheet").
  - Live **search box**.
  - **Theme toggle** button (☀ / 🌙).
  - **Expand all** / **Collapse all** buttons.
- **Body:** 15 collapsible module cards using native `<details>`/`<summary>`.
  - `<summary>` = module title (the clickable header).
  - Inside: concept summary paragraph + command block(s).
  - Each command block: a `<pre><code>` with a **copy button** positioned in its corner.

## Behaviors (vanilla JS)

### Live search / filter
- Input event on the search box filters in real time.
- Match is case-insensitive against each module's full text (title + concept + commands).
- Modules with zero matches are hidden (`hidden` attribute / `display:none`).
- Matching modules are force-expanded so results are visible.
- Matched substrings are highlighted (wrap in `<mark>`).
- Clearing the box restores all modules to their prior collapsed/expanded state.
- A "no results" message shows when nothing matches.

### Collapsible modules
- Native `<details>` elements — keyboard accessible, no library.
- **Expand all** opens every `<details>`; **Collapse all** closes them.

### Copy-to-clipboard
- Each command block has a copy button.
- Click → `navigator.clipboard.writeText(<raw command text>)`.
- On success the button briefly shows "Copied ✓" then reverts (~1.5s).
- Fallback: if `navigator.clipboard` is unavailable (non-secure context such as `file://`
  in some browsers), use a `document.execCommand('copy')` fallback via a temporary
  `<textarea>` so copy still works offline.

### Theming
- CSS custom properties define two palettes (dark terminal default, light K8s-blue).
- Default follows OS via `@media (prefers-color-scheme: light/dark)`.
- Manual toggle sets `data-theme="light|dark"` on `<html>`, which overrides the media
  query, and persists the choice in `localStorage` (read on load).

## Styling

- **Dark (default):** dark background, monospace command blocks, green/cyan accents —
  terminal feel.
- **Light:** white module cards, Kubernetes blue (#326CE5) accents, docs feel.
- Responsive: readable on a laptop and a phone (single column, wrapping command blocks
  with horizontal scroll on overflow rather than breaking commands).

## File Structure

```
cheatsheet.html      # the single deliverable (root)
```

Internally:
```
<!doctype html>
<html>
  <head>
    <style> … CSS variables + both themes + layout … </style>
  </head>
  <body>
    <header> … title, search, theme toggle, expand/collapse … </header>
    <main>
      <details> … Module 0 … </details>
      … Modules 1–14 …
    </main>
    <script> … search, collapse-all, copy, theme … </script>
  </body>
</html>
```

## Accessibility & Quality

- Keyboard: `<details>`/`<summary>` and `<button>` elements are natively focusable.
- Buttons have discernible labels (`aria-label` on icon-only buttons).
- Copy buttons announce success via the visible "Copied ✓" text.
- Color choices keep readable contrast in both themes.

## Verification

Manual acceptance checks (open `cheatsheet.html` in a browser):

1. All 15 modules render with correct titles, concept lines, and command blocks.
2. Search "rollout" → only modules containing it remain, expanded, with matches highlighted;
   clearing restores all.
3. Collapse all → every module closed; Expand all → every module open.
4. Click a copy button → clipboard holds that exact command; button shows "Copied ✓".
5. Toggle theme → palette switches and persists across reload; with no stored choice, the
   page matches the OS theme.
6. Open via `file://` (double-click) with no network → everything works, copy included.
