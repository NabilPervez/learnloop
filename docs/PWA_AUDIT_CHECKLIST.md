# LearnLoop — PWA Audit & Fix Checklist

_Audit date: 2026-09-17 · Scope: entire repo (`index.html`, single commit `cdcbcc3`)_

## Summary

LearnLoop is a well-designed single-file app (vanilla JS, IndexedDB persistence, Backlog → Planner → Execute flow).
It **cannot install or work offline today**. `index.html` references `manifest.json`, `sw.js`, and `icons/icon-192.png`,
but **none of those files exist in the repo**. The failed service worker registration is silently swallowed
(`.catch(() => {})`), so nothing looks broken until someone tries to install the app.

Severity: **P0** = blocks "working PWA" · **P1** = broken/unreliable behavior · **P2** = quality, UX, a11y · **P3** = nice to have

---

## 1. Installability (P0)

| # | Problem | Evidence | Fix | Done |
|---|---|---|---|---|
| 1.1 | `manifest.json` missing (404) | `<link rel="manifest" href="manifest.json">` | Add manifest: `name`, `short_name`, `id`, `start_url: "./"`, `scope: "./"`, `display: "standalone"`, `background_color`/`theme_color: #181A26`, `orientation`, `description` | [ ] |
| 1.2 | No icons at all | `icons/icon-192.png` referenced; `icons/` doesn't exist | Make 192, 512, maskable 512 (safe zone), 180 apple-touch-icon, favicon.svg; declare with `purpose` | [ ] |
| 1.3 | `sw.js` missing, so no offline support and no install on Chromium | `navigator.serviceWorker.register("sw.js")` | Add a service worker (see §2) | [ ] |
| 1.4 | Registration errors hidden | `.catch(() => {})` | Log a warning; register on `load` | [ ] |
| 1.5 | No HTTPS hosting/deploy defined | No deploy config or README | Deploy to GitHub Pages or Netlify; keep relative paths so it works under `/learnloop/` | [ ] |
| 1.6 | No manifest screenshots (Chrome rich install UI) | — | Add `screenshots` (narrow + wide) | [ ] |
| 1.7 | Incomplete iOS meta | Only `apple-mobile-web-app-capable` | Add `mobile-web-app-capable`, `apple-mobile-web-app-title`, 180px touch icon, optional splash screens | [ ] |

## 2. Offline & caching (P0/P1)

| # | Problem | Fix | Done |
|---|---|---|---|
| 2.1 | App shell not cached | Precache `./`, `index.html`, `manifest.json`, icons on `install` | [ ] |
| 2.2 | Google Fonts (Fraunces, Public Sans, Space Mono) fail offline, so fonts fall back and layout shifts | **Self-host** woff2 files and precache them (also removes a third-party privacy request). Otherwise use stale-while-revalidate for `fonts.googleapis.com` and cache-first for `fonts.gstatic.com` | [ ] |
| 2.3 | No cache versioning/cleanup | `CACHE_VERSION` constant; delete old caches in `activate` | [ ] |
| 2.4 | No update flow, so users get stuck on stale builds | Detect a `waiting` worker, then show a "New version — Reload" toast that posts `SKIP_WAITING`, then reload on `controllerchange` | [ ] |
| 2.5 | No navigation fallback | Network-first navigations, falling back to cached `index.html` | [ ] |
| 2.6 | Storage can be evicted (Safari ITP, low disk) | `navigator.storage.persist()` after the first sprint is created | [ ] |

## 3. Data integrity & persistence (P1)

| # | Problem | Evidence | Fix | Done |
|---|---|---|---|---|
| 3.1 | Last edits can be lost: saves are debounced 150ms and never flushed when the app is backgrounded or killed | `persist()` uses `setTimeout` | Flush on `visibilitychange` (hidden) and `pagehide` | [ ] |
| 3.2 | No schema version or migration | `idbGet("state")` assigned straight to state | `state.version` + `migrate()` + validation, falling back to `freshState()` on corruption | [ ] |
| 3.3 | IndexedDB failure is silent, so the app acts like it saved | `console.warn` only | Toast + in-memory fallback warning (e.g. private mode) | [ ] |
| 3.4 | No backup/export; "End sprint" wipes everything | `state = freshState()` | JSON export/import; optionally archive past sprints | [ ] |
| 3.5 | Multi-tab writes clobber each other | Last write wins | `BroadcastChannel` re-sync, or reload state on focus | [ ] |
| 3.6 | Weak IDs | `Math.random` `uid()` | `crypto.randomUUID()` with fallback | [ ] |

## 4. Functional bugs / logic gaps (P1)

| # | Problem | Evidence | Fix | Done |
|---|---|---|---|---|
| 4.1 | Tasks can't be edited or deleted (only repeat or schedule) | `taskCardHtml` | Edit/delete sheet with Undo toast | [ ] |
| 4.2 | An unscheduled completed task returns to the backlog still marked complete | `bindPlanner` title click keeps `isCompleted` | Reset completion, or block unscheduling completed tasks | [ ] |
| 4.3 | Tapping a task title in the Planner silently unschedules it, with no affordance or undo | `.mt-title` click handler | Explicit "Move to backlog" action + Undo | [ ] |
| 4.4 | `openPickFromBacklog` doesn't re-check `isLocked` | Pick handler | Guard `isLocked` in every mutation path | [ ] |
| 4.5 | Focus timer drifts or freezes when backgrounded or the screen is locked: it counts `setInterval` ticks | `remaining--` | Store an `endAt` timestamp and compute remaining from `Date.now()`; persist the active timer across reloads | [ ] |
| 4.6 | Timer can stack intervals (one global `focusInterval`; overlay can open twice) | `openFocusTimer` | Re-entry guard; clear before creating | [ ] |
| 4.7 | "Time's up" goes unnoticed in the background; `AudioContext` created without a user gesture is blocked on iOS | `chime()` | Unlock audio on the start tap; Notification permission + notification + vibrate on completion; Screen Wake Lock during focus | [ ] |
| 4.8 | Days aren't tied to calendar dates, and Execute doesn't auto-select today | `makeSprint` stores only `createdAt` | Derive dates from `createdAt`; default Execute to today; show end date | [ ] |
| 4.9 | No sprint-complete or retro state | — | End-of-sprint summary (minutes done, primary vs secondary) | [ ] |
| 4.10 | `bindBottomNav()` duplicated in each screen binder | Maintainability | Call once from `render()` | [ ] |
| 4.11 | Full `innerHTML` re-render on every tap loses scroll position (planner) and focus | `render()` | Preserve scroll/focus, or do targeted updates | [ ] |

## 5. Accessibility & mobile UX (P2)

| # | Problem | Fix | Done |
|---|---|---|---|
| 5.1 | `maximum-scale=1` blocks pinch-zoom (WCAG 1.4.4) | Remove; keep inputs ≥16px to avoid the iOS auto-zoom | [ ] |
| 5.2 | Clickable `div`s (chips, toggles, sizes, cards, checkbox, menu) aren't keyboard or screen-reader operable | `<button>`, `aria-pressed`, `role="radiogroup"`, labels | [ ] |
| 5.3 | Sheets lack dialog semantics, focus trap, Escape to close, and focus return | `role="dialog" aria-modal` + focus management | [ ] |
| 5.4 | Toast not announced | `role="status" aria-live="polite"` | [ ] |
| 5.5 | Icon buttons named only via `title` | `aria-label` | [ ] |
| 5.6 | With `black-translucent`, the topbar and bottom nav sit under the notch/home bar (only the sheet uses `env(safe-area-inset-*)`) | Safe-area padding on topbar + nav | [ ] |
| 5.7 | No `prefers-reduced-motion` | Disable transitions | [ ] |
| 5.8 | 🔒 emoji is the only lock-state indicator | Add text/aria | [ ] |
| 5.9 | No `<noscript>` | Add | [ ] |
| 5.10 | `--paper-dim` contrast on glass at small sizes unverified | Audit with axe/Lighthouse | [ ] |

## 6. Engineering & delivery (P2/P3)

| # | Problem | Fix | Done |
|---|---|---|---|
| 6.1 | No README/license | README: what it is, run locally, deploy, install | [ ] |
| 6.2 | 1,683-line single file | Split into `styles.css` + `app.js` (ES modules), no build step | [ ] |
| 6.3 | No tests | Unit tests (capacity math, migrations) + Playwright smoke test incl. offline reload | [ ] |
| 6.4 | No CI | GitHub Action: tests + Lighthouse CI budgets | [ ] |
| 6.5 | No CSP | Meta CSP `default-src 'self'` (after moving inline script out) | [ ] |
| 6.6 | No `.gitignore` / `.nojekyll` | Add | [ ] |
| 6.7 | No manifest shortcuts (P3) | `shortcuts`: Add task, Today | [ ] |

---

## Definition of Done: "working PWA"

- [ ] Lighthouse: installable; Performance ≥ 90, Accessibility ≥ 95, Best Practices ≥ 95
- [ ] Installs on Android Chrome, desktop Chrome/Edge, and iOS Safari with correct icon, name, and splash
- [ ] Airplane mode: installed app cold-launches with correct fonts, and every feature works
- [ ] Data survives app kill mid-edit, reload, and an app update
- [ ] New deploy shows the "Update available" prompt, and reload applies it
- [ ] Focus timer is accurate after 10 min backgrounded or locked, and notifies on completion
- [ ] Fully keyboard- and screen-reader-operable
