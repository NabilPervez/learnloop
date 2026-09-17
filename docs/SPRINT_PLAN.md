# LearnLoop — PWA Remediation Sprint Plan

Source: [PWA_AUDIT_CHECKLIST.md](./PWA_AUDIT_CHECKLIST.md). Numbers refer to checklist items.
Cadence: 4 one-week sprints, 1 developer (~20–25 focused hrs/week). Each sprint ends shippable.

---

## Sprint 1: Installable & offline (P0)
**Goal:** The deployed app installs on Android, desktop, and iOS and cold-launches in airplane mode.

| Task | Checklist | Est. |
|---|---|---|
| Icon set (192, 512, maskable 512, apple 180, favicon.svg) | 1.2 | 2h |
| `manifest.json` (id, scope, start_url, standalone, colors, icons) | 1.1, 1.6 | 1h |
| Self-host fonts (woff2); drop the Google Fonts link | 2.2 | 2h |
| `sw.js`: versioned precache, activate cleanup, network-first navigation + offline fallback, cache-first static | 1.3, 2.1, 2.3, 2.5 | 4h |
| Register SW on `load` with logged errors | 1.4 | 0.5h |
| iOS meta tags + safe-area padding on topbar/bottom nav | 1.7, 5.6 | 2h |
| Remove `maximum-scale=1` | 5.1 | 0.25h |
| Deploy to GitHub Pages (`.nojekyll`) or Netlify; verify subpath | 1.5, 6.6 | 1.5h |
| README (run with `npx serve`, deploy, install) | 6.1 | 1h |
| **Verify:** Lighthouse installable; airplane-mode cold start on 3 platforms | DoD | 2h |

**Exit:** The install prompt appears, the app opens offline with the right fonts, and Lighthouse shows no PWA errors.

## Sprint 2: Durable data & safe updates (P1)
**Goal:** User data is durable and versioned, and releases roll out cleanly.

| Task | Checklist | Est. |
|---|---|---|
| Flush pending save on `visibilitychange`/`pagehide` | 3.1 | 1h |
| `state.version` + `migrate()` + validation/corruption fallback | 3.2 | 3h |
| Surface IndexedDB failures | 3.3 | 1.5h |
| `navigator.storage.persist()` | 2.6 | 0.5h |
| SW update flow (waiting → toast → SKIP_WAITING → reload) | 2.4 | 3h |
| JSON export/import in sprint menu | 3.4 | 3h |
| Multi-tab sync via `BroadcastChannel` | 3.5 | 2h |
| `crypto.randomUUID()` IDs | 3.6 | 0.25h |
| Split into `styles.css` + `app.js` modules; update precache list | 6.2 | 4h |

**Exit:** Killing the app mid-edit loses nothing, old saves migrate, and a new deploy prompts the user and applies.

## Sprint 3: Logic fixes & reliable focus timer (P1)
**Goal:** Every flow is predictable, and the timer works in real-world conditions.

| Task | Checklist | Est. |
|---|---|---|
| Timestamp timer (`endAt`), re-entry guard, persist active timer | 4.5, 4.6 | 3h |
| Audio unlock on gesture; SW `showNotification` on completion; vibrate; Wake Lock | 4.7 | 4h |
| Edit/delete task sheet with Undo | 4.1 | 3h |
| Explicit "Move to backlog" + Undo; reset completion | 4.2, 4.3 | 2h |
| Lock guards on all mutation paths | 4.4 | 1h |
| Calendar dates per day; Execute defaults to today | 4.8 | 3h |
| Sprint-complete summary | 4.9 | 3h |
| Render cleanup (single nav bind, preserve scroll/focus) | 4.10, 4.11 | 2h |

**Exit:** A 10-minute backgrounded timer ends on time and notifies, and no destructive action is hidden or lacks undo.

## Sprint 4: Accessible, tested, guarded (P2/P3)
**Goal:** Meets accessibility standards, and CI prevents regressions.

| Task | Checklist | Est. |
|---|---|---|
| Buttons + ARIA instead of clickable divs | 5.2, 5.5, 5.8 | 4h |
| Dialog semantics, focus trap, Escape, focus return | 5.3 | 3h |
| Live-region toast, reduced motion, `<noscript>`, contrast | 5.4, 5.7, 5.9, 5.10 | 2h |
| Unit tests (capacity math, migrations) | 6.3 | 3h |
| Playwright E2E incl. offline reload | 6.3 | 4h |
| GitHub Actions: tests + Lighthouse CI; auto-bump `CACHE_VERSION` | 6.4 | 2h |
| CSP; manifest shortcuts + screenshots | 6.5, 6.7, 1.6 | 2h |

**Exit:** The checklist's Definition of Done is fully ticked, and CI is green.

---

## Risks
- **iOS:** notifications only work for *installed* PWAs (iOS 16.4+), and background timers can't fire exactly. Mitigation: timestamp timer + catch-up on resume.
- **Stale caches:** forgetting to bump `CACHE_VERSION` ships old code. Automate the bump in CI.
- **File split (Sprint 2):** do it before the Sprint 3 logic changes, and smoke-test right after.
