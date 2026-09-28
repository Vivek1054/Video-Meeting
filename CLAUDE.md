# Video Meetings prototype (Teloz / MCM)

Front-end-only prototype of a "Video → Meetings" module: schedule / start / join meetings, a live meeting room, recordings, and a demo user switcher (Admin / Location Admin / Manager / Agent). Everything runs from mock data in the browser. No backend, no build step, no package.json, not a git repo.

## Stack (only what exists)
- HTML + CSS + vanilla JS (`script.js` is one IIFE, `"use strict"`).
- Bootstrap 5.3.3 (`vendor/bootstrap.min.css` with the reboot section removed, `vendor/bootstrap.bundle.min.js`) + Bootstrap Icons (`vendor/bootstrap-icons/`).
- MediaPipe selfie segmentation (`vendor/mediapipe/`, loaded lazily for background blur/replace).

## Files
- `index.html` – the only page. Top navbar, left icon rail, Meetings sidebar, main views, every modal, the live room (`#liveModalOverlay`) and its dialogs.
- `style.css` – all custom CSS, sectioned by `/* ====== NAME ====== */` (navbar, app shell, meeting cards, modals, pre-join lobby, live room, drawer, calendar, recordings, role switcher, context menu, toasts, responsive).
- `script.js` – all logic, sectioned by `N. TITLE` comments (search the title): 0 Utilities (`$`, `$$`, `on`) · 1 Mock data (locations/teams/`USERS`/`let ME`) · 2 State · **2b Role-based access control** (`PERMISSIONS`, `can`/`authorize`, `visibleX()`, the user switcher) · **2c Routes + pages** (hash router, Dashboard/Directory/Analytics/Settings + Administration pages) · 3 Toasts · 4 Modal helpers · 5 Dark mode/clock · 6 Navigation · 7 Meeting cards · 8 Calendar · 9 Toolbar (search/filter/sort) · 10 Card actions · 11 Context menu (`MENU_CONFIG`) · 12 Schedule modal · 13 Join/pre-join · **13c Video effects engine** · 14 Start Meeting (pre-join lobby) · 14b Meeting tab (`?view=meeting`) · **15 Live room** · 16 Details drawer · 17 Reschedule/cancel · 18 Invite · 19 Recordings · 20 Demo data toggle · 21 Notifications · 22 Command palette · 23 Global render · 24 Init.
- `vendor/` – third-party files, never edit.

## Where a feature lives
| Feature | HTML (`index.html`) | JS section |
|---|---|---|
| Meeting lists / cards / calendar | `#view-upcoming/ongoing/invited/past`, `.meeting-card` | 7, 8, 9, 10, 11 |
| Schedule / edit / duplicate | `#scheduleModalOverlay` | 12 |
| Join by ID, pre-join | `#joinModalOverlay`, `#prejoinModalOverlay` | 13 |
| Start Meeting popup (camera preview, background, enhancements) | `#startModalOverlay` | 14, 13c |
| Live room (tiles, chat/Q&A/participants, More Actions, end/leave) | `#liveModalOverlay`, `#endMeetingConfirmOverlay` | 15 |
| Live-room side panel (`live.panel` = `chat/participants/qa/more/ai-notes`): Chat, More (grouped; Meeting Intelligence Live/Demo + Demo controls), **Notes & Coach** = Meeting Intelligence (Transcript / Notes / Playbook / Live Coach). Separate `aiNotes.live` / `aiNotes.demo` states, one demo scheduler, `MeetingIntelligenceService` (rule-based / demo / no AI provider); notes saved per meeting code in `mcm-notes-v1` (live only) | `#livePanel`, `#liveMoreMenu`, `#liveAiPanel` | 15 (`renderLivePanel`; "Notes & Coach" block after captions) |
| Effects (blur/replace, lighting, appearance, sliders, mic clean-up) | `#lobbyFxPanel`, `#dlgEffects`, `#liveFxPanel` | 13c |
| Details drawer, reschedule, cancel, delete, invite | `#detailsDrawerOverlay` etc. | 16–18 |
| Recordings | `#view-recordings`, `#recordingPreviewOverlay` | 19 |
| Roles + user switcher | `#roleSwitchBtn`, `#roleMenu`, `data-perm` attributes | 2b |
| Dashboard / Directory / Analytics / Settings + Administration pages (hash routes `#/dashboard` etc.) | `#view-page`, `#pageBody` | 2c |
| Notifications, Ctrl+K search, dark mode | navbar | 21, 22, 5 |

Video Meetings, Dashboard, Directory, Analytics and Settings (+ Administration) are real, all RBAC-scoped. The left rail (Phone, Chat, Agent Chat, Inbox, Campaign, Tasks, Calendar) is still placeholder.

## UI patterns
- Modals: `.modal-overlay` + `openModal(id)` / `closeModal(id)` (uses `hidden` + `.open`), close buttons use `data-close-modal`. Live-room dialogs (`#dlgEffects`, `#dlgInfo`…) use `openLiveDialog`.
- Toasts: `toast(message, "error")` (custom `.toast`, click-through).
- Dropdowns/tooltips: Bootstrap dropdowns (role menu, live-room menus, lobby "Adjust" panel); locked controls and effect toggles get Bootstrap tooltips via `data-rbac-tip`. Card ⋮ menu is custom (`#meetingActionMenu`, `MENU_CONFIG`).
- Tiles/tabs/forms use Bootstrap classes plus app classes (`live-*`, `lobby-*`).
- Views are switched with `switchSidebarView`; event delegation is used on cards and the live room.

## Styling rules
- Prefer Bootstrap utilities/components; add minimal custom CSS. Bootstrap sits in `@layer bootstrap`, plus `@layer rbac` (only `.rbac-hidden`).
- Sizes in `rem` (`html { font-size: 90% }`); never px for new UI. Colours are CSS variables in `:root`; ONE theme for the whole app: `[data-theme="dark"]` on `<html>`, saved as `mcm-dark-mode`, flipped by `applyTheme()` (navbar `#darkModeToggle` and the meeting header `#liveThemeBtn`). The live room follows it: its colours are `--live-*` tokens in `.live-room` (dark values) with the light values in "LIVE ROOM: LIGHT THEME" at the end of `style.css`; add new room colours as tokens, not literals.
- Bootstrap's reboot is removed, so `[hidden]` is not enforced: an element that has `display:` in CSS needs its own `.x[hidden]{display:none}`.
- Bootstrap `.ratio` stretches direct children; keep overlays outside it.

## JavaScript rules
- Keep the single-IIFE, function-per-feature style. State lives in `state` (section 2), saved by `saveState()` to localStorage (`mcm-video-meetings-state-v1`; others: `mcm-dark-mode`, `mcm-video-fx-v1`, `mcm-video-fx-image-v1`).
- Mock data: section 1 (`USERS`, `DEMO_ACCESS`, `buildDemo*`), toggled by the "Demo data" switch (section 20).
- Reuse `$`, `$$`, `on`, `toast`, `openModal`; don't add libraries.

## Roles (sections 2b–2c)
4 roles — Admin (global) > Location Admin (their `locationId`) > Manager (the team(s) they manage) > Agent (own/assigned) — driven by WHO is signed in, not a role picker: the pill beside "Meetings" (`#roleSwitchBtn`/`#roleMenu`) is a **demo user switcher** over `USERS` (10 people across `LOCATIONS`/`TEAMS`; 8 marked `switcher: true` appear in the menu), and `setCurrentUser()` reassigns `let ME`. One `PERMISSIONS` table (`grant(admin, location_admin, manager, agent)` → scope lists: `all/location/team/own/assigned/invited/shared/delegated/self`) + `canUser(user, perm, item)` / `can(perm, item)` / `authorize(perm, item)`; controls carry `data-perm` (+ `data-perm-mode="lock"` for dimmed-with-tooltip), every action handler also calls `authorize()`, and every list/picker/search/notification/palette result is built from `visibleMeetings()/visibleRecordings()/visibleUsers()` (filter first, render after — never filter a rendered list). Dashboard/Directory/Analytics/Settings + Administration are real pages behind a hash router (`ROUTES`/`SETTINGS_PAGES` in 2c), each declaring the permission it needs; `renderRoute()` re-checks on every navigation and every user switch, so an unauthorized address (typed or left over after switching user) always renders "Access denied", never the page's data. The Meetings sidebar/cards are identical for every role, only the data + available actions change. New permission → add to `PERMISSIONS`, add `data-perm`, call `authorize()`. New page → add to `ROUTES`/`SETTINGS_PAGES` with its `perm`.
This is a client-side-only prototype: every check here is UI convenience, not security — a real deployment must re-check role + location + team + ownership on the server for every API call.

## Must not break
Sidebar/view switching, fixed shell with independent scrolling (page itself never scrolls), all modals, card ⋮ menu, schedule/edit/reschedule/cancel/delete/invite, join & Start Meeting (opens a new tab via `?view=meeting`), live room controls (mic/cam/share/chat/Q&A/reactions/hand/whiteboard/More Actions), End vs Leave dialog by role, role switching, recordings, demo data toggle, dark mode, responsive layout, pre-join popup fitting without scrolling.

## Working rules (fast by default)
Default flow: **Understand → Locate → Edit → Quick check → Done.**
- Start at the file/selector/function I name. Otherwise use this map, Grep, then Read only a small range. Don't scan the project, don't re-read files, don't open unrelated files or `vendor/`.
- Make the smallest edit that works, directly in the file (Edit tool). No patch scripts, temp tooling or new files for small changes. Reuse existing classes/functions/Bootstrap.
- Don't refactor, redesign or "improve" anything I didn't ask for; stop when the task is done. Don't ask questions when the request is clear.
- Simple request: no long planning. Complex request (video/audio/canvas/new feature): a short plan, then edit.

## Testing rules (never escalate on your own)
- **Default:** quick sanity check only (re-read the edited spot, class names/markup, `node --check script.js` for JS). No test suites, no browser automation, no screenshots, no servers, no fixing old tests.
- **"test once"** → one targeted smoke test of the changed feature. **"full test"** → the relevant complete suite. **"regression test"** → the existing relevant regression suites. **"screenshots"** → only then take them.
- Old Playwright suites live outside the repo in `%TEMP%\pwtest` (`rbac_test.js`, `fx_test.js`, `sp_*.js`); use them only when asked. Start/stop test servers only for a requested test.
- If a prompt contains a long testing checklist, still do one smoke test unless it says "full test".

## Priority when rules conflict
1. My current request 2. Not breaking existing functionality 3. This file 4. Existing conventions 5. Smallest safe change.

## Keeping this file current
Update only for big changes (new page/major feature, file responsibilities, dependency, role behaviour). Not after small UI edits. Keep it short.
