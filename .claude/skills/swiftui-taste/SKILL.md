---
name: swiftui-taste
description: Use when designing, restyling, reviewing, or polishing any SwiftUI/AppKit UI in mac-tmux-kit (menu-bar popover, Dashboard, Settings, command palette, console, cheatsheet), or when iterating on look from a screenshot. Enforces native-macOS taste, consistency locks via the Theme tokens, complete empty/error/loading states, motivated motion, accessibility, and an anti-slop banlist. Distilled for SwiftUI from the web-oriented "taste-skill" anti-slop framework — none of that repo's React/Tailwind/GSAP specifics apply here.
---

# SwiftUI Taste — mac-tmux-kit

Native-macOS-first UI discipline for this repo. Every rule is **contextual**: read what the surface is for, then pull only what fits. The single source of truth for color/type/metrics is `MacTmuxKit/Design/DesignTokens.swift` (`enum Theme`), whose palette is resolved **live from the user's Ghostty theme** at launch (`GhosttyTheme.swift`), falling back to Flexoki.

## 0. Before touching a view

State a one-line **design read**: *"This is the \<surface> for \<job>; it should feel \<native macOS / terminal-adjacent>."* Surfaces:
- **Menu-bar popover** (`Features/MenuBar`) — fast switcher, 360pt wide, glanceable.
- **Dashboard** (`Features/Dashboard`) — 3-column NavigationSplitView browser.
- **Settings** (`Features/Settings`) — grouped Forms.
- **Command palette / Console / Cheatsheet** — focused utility windows.

If a change diverges (e.g. new accent, new layout family), say so and why before editing.

## 1. Consistency locks (the core of "taste")

- **One accent, locked.** Use `Theme.accent` (live Ghostty green); `Theme.accentSoft` for selection/row fills. Never introduce a second hue as decoration. `Theme.ghostty.blue` is the only sanctioned alternate, and only if a deliberate decision.
- **Status colors are state, never decoration.** `Theme.attached / success / warning / danger` only signal real state (attached session, ok/warn/error). Don't use red/green/yellow for flair.
- **One radius per element class.** `Theme.Radius.card` (6) for cards/tiles, `.row` (7) for list rows, `.panel` (12) for windows/panels. Don't mix scales within a class.
- **One type system.** Use `Theme.Font.*` roles. Technical labels (counts, ids, pids, dimensions — `3w`, `%5`, `pid 412`) are **monospaced** via `Theme.Font.metric / metricSmall / terminal`. Don't hand-pick `.system(size:)` in views.
- **Theme never inverts.** The app follows one resolved Ghostty theme. Don't sandwich a hardcoded light/dark section between others.

## 2. Native-first rule

Only the **accent** and the **monospaced technical-label voice** are opinionated. Everything else rides macOS:
- Text tiers → `.primary / .secondary / .tertiary`, not hardcoded grays.
- Chrome/surfaces → system materials (`.regularMaterial`, vibrancy sidebar/menu-bar), not custom blur.
- Borders → `Theme.hairline` (luminance, 0.08 opacity); hovered custom rows → `Theme.hoverFill` (0.07).
- **Never hardcode hex or gray in a view** — go through `Theme`, or the GUI stops tracking the user's terminal theme.

## 3. Complete interaction states (don't ship the happy path only)

- **Empty vs error must be distinguishable.** A genuine "nothing here" uses `EmptyStateView` (icon + title + actionable subtitle, e.g. "Create one with the +"). A failure (can't reach the tmux server, binary missing) must read as an **error**, not as emptiness. (This is exactly the "No tmux sessions" ambiguity that masked a real bug — see `AppState.refresh()` / `TmuxError`.)
- **Loading:** use `isLoading`; prefer a quiet inline `ProgressView().controlSize(.small)` in the header, or a shape-matching placeholder for lists. Don't flash a full-window spinner on every refresh.
- **Optimistic feedback:** mutating actions flash a `ToastView` via `AppState.run(success:)`. Keep success toasts short; surface failures both inline (`statusMessage`) and as a toast.

## 4. Motion — motivated, native, accessible

- Default to `.spring` / `.easeOut`; never `.linear` for UI feedback.
- **Gate every animation on reduce-motion:** `@Environment(\.accessibilityReduceMotion)` (Dashboard already does `reduceMotion ? nil : .easeOut(...)`).
- Motion must communicate **hierarchy / feedback / state change**. If you can't name its job in one sentence, drop it. No infinite ambient loops.

## 5. Accessibility & tactile detail

- **Contrast:** verify accent/status text and buttons stay legible over their background and over the live terminal canvas (`Theme.terminalBackground`). Target WCAG AA.
- **Tactile press:** interactive rows/buttons respond on press (`.scale(0.98)` / borderless hover fill), matching existing `SessionRow`/`SessionSidebarRow`.
- **Icons:** SF Symbols only, consistent weight/size. Never hand-roll SVG/Path glyphs. No emoji in UI text.

## 6. Anti-slop banlist (macOS translation)

❌ AI-purple / invented gradients · decorative glassmorphism everywhere · hardcoded hex or gray in views · a second accent hue · status colors as decoration · emoji in UI · hand-rolled icon paths · eyebrow-label spam on every section · mixed radius scales · full-window spinner on routine refresh · empty state that's really an error.

✅ Theme-driven color · system materials with purpose · monospaced voice for technical labels · one accent locked · spring + reduce-motion · SF Symbols · actionable empty states distinct from errors.

## 7. audit-first when iterating from a screenshot

The maintainer iterates on look via screenshots. When handed one:
1. **List concrete issues first** (spacing rhythm, hierarchy, contrast, inconsistent radius/type/accent, missing state) — don't restyle blind.
2. Propose the **minimal** diff that fixes them; touch only the relevant view.
3. Re-check against the pre-ship list, then build via `./scripts/run.sh` (the only correct build path — preserves the Accessibility grant).

## 8. Pre-ship checklist (mechanical)

- [ ] No hardcoded hex/gray; all color via `Theme`
- [ ] One accent; status colors only signal state
- [ ] Radius from `Theme.Radius.*`, one scale per element class
- [ ] Type from `Theme.Font.*`; technical labels monospaced
- [ ] Empty / error / loading all handled and distinguishable
- [ ] Animations gated on `accessibilityReduceMotion`, each motivated
- [ ] SF Symbols only, consistent weight; no emoji
- [ ] Contrast holds over both app chrome and the terminal canvas
- [ ] Builds clean via `./scripts/run.sh`
