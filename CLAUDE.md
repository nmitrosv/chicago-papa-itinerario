# chicago-papa-itinerario

Two static, self-contained HTML pages hosted on GitHub Pages (no build step):

- `index.html` — day-by-day trip itinerary (Chicagoland).
- `menu-papa.html` — home-cooking meal plan + shopping checklists, cross-linked with the itinerary.

Both pull from the same design tokens (`--aegean`, `--gold`, `--olive`, `--ink`, `--line`, etc.) defined at the top of each file's `<style>`. When editing one, match the values in the other rather than inventing new colors/spacing.

## UI/UX lessons learned (from a 2026-07 audit)

These are mistakes actually made in this repo — read before touching CSS/layout here or in similar static-page projects.

1. **Every standalone HTML page needs its own full boilerplate — always.** `menu-papa.html` was originally authored as an Artifact-tool fragment (the Artifact publisher auto-wraps content in `<!doctype html><head>...<body>`), then later committed straight to the repo for GitHub Pages. GitHub Pages does **not** add that wrapper — the file was missing `<!DOCTYPE html>`, `<html>`, `<head>`, `<meta charset>`, and critically `<meta name="viewport" content="width=device-width, initial-scale=1">` entirely. Lacking the viewport tag is a real cross-browser risk even if one browser's fallback heuristics happen to render acceptably. **Whenever a page moves from "rendered inside another wrapper" (an Artifact, an iframe, a CMS template) to "served directly," re-verify it still has the meta essentials that the other mechanism was silently providing.**

2. **A horizontally-scrolling flex/grid nav needs a real trailing spacer element, not just container `padding`.** iOS Safari/WebKit has a long-standing quirk where the padding on the far/trailing edge of a scrollable container gets clipped once scrolled to the end — the last item looks flush against the viewport edge with no breathing room, like it's cut off. This is exactly what happened with the "📅 Itinerario completo" pill in `menu-papa.html`'s nav. Container padding on a scrollable axis is not reliable at the scroll end on WebKit. **Fix: add an actual empty flex child (e.g. `<span class="nav-end-spacer">`) with a fixed `flex: 0 0 Npx` after the last real item** — a real box in the flow contributes properly to `scrollWidth` regardless of the padding-clipping quirk.

3. **Don't shrink tap targets below ~40px just to cram more nav items into a narrow viewport.** A prior pass to fix visual crowding shrank nav-pill padding/font enough that touch targets dropped to ~24-26px tall — well under the ~44pt (iOS)/48dp (Material) accessibility guidance. Reducing spacing is fine; reducing the *tappable height* below the platform minimum is not an acceptable trade for fitting more on screen. If items don't fit, let the row scroll (with fix #2 above) rather than sacrificing target size. Always set an explicit `min-height` on tappable nav pills/buttons rather than relying on font-size + padding math to land in a safe range.

4. **Keyboard focus styles must exist on every page of a multi-page project, not just one.** `index.html` had `:focus-visible` outline rules; `menu-papa.html` didn't. When two pages share a design system, audit both for the same baseline a11y rules (focus outlines, contrast, alt text) — it's easy to add something once and forget to port it when a second page is created later.

5. **When a user reports something looks "cut off" or "misaligned" at an edge, suspect scroll-container edge-clipping first**, especially on a horizontally scrollable row on iOS — it's a narrower, more specific diagnosis than generic "spacing is off," and shrinking everything (lesson #3) is the wrong fix for it.

## Workflow

This repo gets worked on across many separate Claude Code sessions (the user runs several in parallel/sequence). Because of that:

- **Always tell the user explicitly when work is done and ready** — don't leave them to ask "ready?"/"is it live?". After finishing a change (and especially after merging to `master`, since that's what GitHub Pages serves), send a clear done/ready signal summarizing what shipped.
- If you claimed something was published, proactively confirm the GitHub Pages deployment for that commit actually succeeded (check the `pages build and deployment` workflow run for the commit SHA) rather than assuming the push alone means it's live.
- Before merging to `master`, fetch it first — other sessions land commits here often, and `master` may have moved since your branch started. Diverged history is normal in this repo, not a sign something went wrong.
