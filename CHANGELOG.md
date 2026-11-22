# Changelog

All notable changes to **QA Trainee Practice Lab** (Buggy Threads Co.) are documented here.
Versions follow `MAJOR.MINOR.PATCH`. The in-page version label is shown in the header.

## [1.8.1] - 2026-11-22

### Added
- **Hard mode is locked.** It opens in one of two ways: confirm every bug in Easy mode, or enter the access code.
  Switching back from Hard to Easy is always free. After Easy is completed, Hard unlocks automatically.
- **Write your own bug.** The report form's *Which bug is this report about?* list now has
  **"Other: my own bug (not in the list)"** with a name field. The report is scored for quality and saved
  with an `own:` marker. It does not award rating points, because it cannot be matched to a planted bug.

### Notes
- The access code is stored as a SHA-256 hash. This is a gate for learners, not security:
  anything that runs in the browser can be bypassed through DevTools.
- To change the code, compute a new hash and replace `HASH` in the `bt-v181` script:
  - Linux/macOS: `printf 'newcode' | sha256sum`
  - PowerShell: `(Get-FileHash -InputStream ([IO.MemoryStream]::new([Text.Encoding]::UTF8.GetBytes('newcode'))) -Algorithm SHA256).Hash.ToLower()`

## [1.8] - 2026-11-22

### Added
- **Login with a real demo account:** `qa@buggythreads.com` / `Passw0rd!` (shown under the form).
  Wrong or empty credentials now show an error (`Invalid credentials`, validation messages), so the negative path can be tested.
- **New hint flow.**
  - First request: one free, random hint (once, ever) for a bug that is not found yet.
  - Every next request: a quiz of 5 random manual-testing questions (pool of 20, answer order shuffled each time).
    4 of 5 correct unlocks one random hint.
  - One global **Get a hint** button replaces the per-bug hint buttons.
  - In Hard mode a hint shows only the first words of the full hint.

### Changed
- The SQL-injection bug (BUG-AUT-01) now triggers on a real `' OR '1'='1` pattern instead of any username containing `admin`.
  Plain `admin` / any password no longer logs in.
- The login form is cleared after a successful login, and the SQL-injection placeholder spoiler is removed.
- Rating points are now 10 per confirmed bug. The quiz no longer awards points, so points cannot be farmed by repeating it.

### Removed
- **QA Control Center -> Project Files tab** and its embedded project archive (README, `server.js`, `mock-api.js`, tests,
  schemas, workflow). It listed every planted bug with its ID and made the answers public.
  The page is about 40 KB smaller. The Flaky Lab text now points to the GitHub repository.
  Keep the automation project in the repository as separate material.

## [1.7] - 2026-11-22

### Fixed (from tester feedback)
- **Bug overlay showed over other windows and side panels.** Overlay labels had `z-index: 9999` and appeared above modals and drawers,
  and they printed the exact bug ID. The overlay is now outline-only and applies only to the topmost open panel.
- The overlay no longer shifts positioned elements (for example the SALE badge).
- **Bugs could be found by accident.** Previously a bug was marked found by a click, by a manual checkbox,
  or by the first word of a report title. Now:
  - a bug that triggers is only **Triggered** (seen);
  - it becomes **Confirmed** only through a report: pick the bug in the report form, the bug must already have been
    reproduced in the app, and the report must score at least 60;
  - the manual checkbox is disabled.
- **Logout:** BUG-AUT-07 (cart survives logout) now counts only when the cart is not empty.
  The header shows `Guest User` after logout, so the logged-in state is visible.

### Added
- **Easy / Hard modes.**
  - Easy: dashed outline overlay (no bug IDs), full hints.
  - Hard: no overlay, bug titles masked (`Bug #NN`), spoiler toasts and placeholders removed.
- **Leaderboard and player count** (local by default). Nickname, top 20, points.
  Set `BACKEND_URL` in the `bt-v17` script to a Firebase Realtime Database URL to share the leaderboard and count players across browsers
  (database rules must allow read/write on `/players`).
  Points are calculated in the browser and can be forged, so treat the leaderboard as unofficial.
- `Reset` keeps the player id, nickname and difficulty.

## Upgrade notes

- Progress saved by older versions is kept. Bugs that were already found stay confirmed.
- New `localStorage` keys: `bt_seen`, `bt_hints`, `bt_hint_free`, `bt_quiz`, `bt_diff`, `bt_hard_unlocked`, `bt_pid`, `bt_nick`.
  `Reset` clears everything except `bt_pid`, `bt_nick` and `bt_diff`.
- If the repository contains the Playwright project (`tests/`, README), update it for the new behavior:
  visual bugs now become **Triggered**, not **Found**, on click; the manual checkbox no longer confirms a bug;
  login now rejects wrong credentials; the Project Files tab no longer exists.

## Verification

Automated checks run against v1.8.1 by executing the page scripts in jsdom: **60 of 60 passed, no runtime errors.**
They cover login, logout, report confirmation, own-bug reports, the hint quiz, Hard-mode lock, overlay layering,
leaderboard, reset, and the 10 planned SQL training queries.
Not covered: visual appearance, mobile layout, real-browser clicks, the API runner, the Jira board, Flaky Lab, checkout and coupons.

## Known limitations

- **Not available offline.** The page loads Tailwind (CDN), Font Awesome, Google Fonts and one Unsplash image from the internet.
  Without a connection, or behind a filtering network, it renders unstyled.
- **The report evaluator is heuristic.** It checks length and numbered steps, so a meaningless report can still confirm a reproduced bug.
  A rubric-based evaluator tied to an action log is planned.
- Everything runs in the browser: ratings and the Hard-mode code can be bypassed with DevTools.

## Roadmap

- `bugOn(id)` per-bug switches, seeded random bug sets per run ("Random Run"), and a larger bug pool built from templates.
- Action log with a site-coverage map and a report rubric.
- SQL and API training tracks (10 tasks each).
- Self-contained build (compiled CSS, local fonts and icons) for offline use.
- Shared leaderboard, daily challenge, and a teacher mode.
