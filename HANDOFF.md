# Reloadly Operational Performance OS — Project Handoff

## What this is
A single-file web app ("Reloadly Operational Performance OS") that combines **task management**, **OKR tracking**, and **enterprise performance reviews** for Reloadly's ~23-person team. Built and maintained by Dren Lubovci (COO).

## Where it lives
- **Live site:** https://reloadly-tasks.github.io/task-manager/ (GitHub Pages)
- **Repo:** https://github.com/Reloadly-tasks/task-manager
- **Editor URL:** https://github.com/Reloadly-tasks/task-manager/edit/main/index.html
- **File:** `index.html` — one file, ~245 KB / ~1,796 lines. All CSS and JS inline. No backend, no build step.
- **Latest commit:** `50b61ec`

## Architecture (important for editing)
- **Everything is in `index.html`.** Styles in a `<style>` block, all logic in one `<script>` block at the bottom.
- **Persistence:** browser `localStorage` only. No server. Data lives in the user's browser.
- **Deploy = commit.** Editing the file on the `main` branch and committing auto-publishes via GitHub Pages. CDN propagation takes ~30–60 seconds. Hard-refresh (Cmd/Ctrl+Shift+R) to see changes.

### Key localStorage keys
`rl_tasks_v4` (tasks), `rl_okr_links`, `rl_okr_active_quarter`, `rl_okr_Q*_YYYY` (OKRs), `rl_perf` (reviews), `rl_perf_view`, `rl_profile_extras_v1` (profile skills/manager/bio), `rl_log_v1` (activity log), `rl_archived`, `rl_pwd_store`, `rl_user_email`, `rl_theme`.

### Data model (critical — easy to get wrong)
- **Tasks** are in a global `data` object **keyed by department**: `data.Management`, `data.Operations`, `data.Finance`, `data.Engineering`, `data.Sales` (each an array). `DEPTS` = those five names. `CFG` holds per-dept `{p: prefix, col: color}` (e.g. prefix "OPS" → IDs like "OPS-001").
- **Task fields:** `id`, `name` (NOT "title"), `assignee` (a name string — can be first name "Dren" OR full "Dren Lubovci"; this is the field now labeled **"DRI"** in the UI), `priority`, `status` (`Done` / `In Progress` / `Not Started` / `Blocked` / `Under Review`), `due`, `startDate`, `progress`, `notes`, `lastUpdated`, `createdAt`.
- **OKR links** stored as `{ krId: [taskId, ...] }` (KR → tasks, NOT task → KRs).
- **Reviews** keyed in `rl_perf`; `reviewee` is stored as an **email** (e.g. `dlubovci@reloadly.com`), so matching review→tasks requires mapping email → name via `USER_REGISTRY`.
- `currentUser = {name:'Dren Lubovci', dept:'Management'}`.

### Useful global functions
`render()` (task board), `renderMyTasks()`, `renderOKR()`, `renderPerf()`, `renderOrgTree()`, `loadData()`, `autoSave()`, `nextId(dept)`, `logEntry(msg, type)`, `toast(msg, type)`, `esc()`.

## Features currently live
- **Task boards** per department (status/priority/avg-progress charts, inline-editable rows).
- **My Tasks** view with inline add-task form (self-assigns to current user).
- **OKR Tracker** (quarterly objectives + key results, progress, confidence, task-linking; objective % = equal-weighted average of its KRs' progress).
- **Performance module** (10 tabs: Overview, Team Health, Calibration, Talent Matrix 9-box, Reviews, Feedback, My Profile, Manager View, My History, Org Tree). Reviews have an 8-stage workflow (Draft→…→Locked), auto-pulled evidence from the task board, threaded comments with @mentions, action items, and a competency scorecard.
- **Org Tree** with full 23-person reporting hierarchy + Talent Directory (skill chips).
- **Activity Log** (filterable/searchable audit trail, CSV export), **Reminders**, **User Management**, dark/light theme, XLS export.

## What was done in the most recent sessions
1. Fixed a crash where opening a review threw `loadTasks is not defined` — rewrote the evidence section to use the real `data[dept]` model (commit `0e05c72`).
2. Profile edits (skills/manager) now re-render the directory on modal close (`08598d5`).
3. Added an ⓘ info tooltip next to each objective % explaining the calculation (`2216165`).
4. Four task-management changes (`50b61ec`):
   - Notes hover-expand (hovering a note widens it into a 300px popover; `title` tooltip fallback).
   - Add-task form added to the My Tasks view (self-assigns to current user).
   - OKR "Link Tasks" picker now only lists tasks that have a DRI assigned.
   - Renamed "Assignee" → "DRI" everywhere (form label, board header, export header, XLS column).

## Known issues / housekeeping
- **Two test tasks to delete:** `MGT-005` "Verify add-from-mytasks works" and `MGT-004` "George Misses" (created during testing). Delete uses a native `confirm()` dialog.
- **Cosmetic:** skill/DRI chips render with a leading space if you type a space after a comma. A `.trim()` would fix it.
- **Not built (need a backend):** real AI performance intelligence, RBAC/SSO/MFA, server-side audit/analytics, AI automation.

## How to make changes (the working method)
Because it's a 245 KB single file, edits are done by: fetch the raw file → apply targeted string replacements → validate JS syntax (`new Function(...)`) → paste the whole file into the GitHub web editor → commit → wait ~55s → hard-refresh and verify on the live site. Each commit redeploys the whole site.

If working from a desktop environment (e.g. Claude Cowork), the more natural workflow is: `git clone` the repo, edit `index.html` directly, `git commit && git push`. GitHub Pages will republish automatically.

## Working with this project from Claude Cowork

**Recommended local workflow:**
1. `git clone https://github.com/Reloadly-tasks/task-manager.git`
2. Open `index.html` in your editor
3. Use Find & Replace for targeted changes (the file is large but flat — one HTML, one `<style>`, one `<script>`)
4. After editing, validate syntax: open `index.html` in a browser; if the page renders and no console errors, you're good. For programmatic validation, run `node -e "new Function(require('fs').readFileSync('index.html','utf8').split('<script>')[1].split('</script>')[0])"` — should exit cleanly.
5. `git add index.html && git commit -m "..." && git push`
6. Wait ~55s, hard-refresh the live site, verify.

**Things that bite you when editing:**
- The file has been deduplicated and there should be exactly ONE `var SCORECARD_DEF =` and ONE `<!DOCTYPE html>`. Re-check these after any large edit — accidentally doubling the file is the most common failure mode.
- Tasks use `name` not `title`, `assignee` not `owner`. Reviews store reviewee as email; tasks store assignee as name. Map between them via `USER_REGISTRY`.
- OKR links go `krId → [taskIds]`, not the other way.
- localStorage is per-browser. Different browsers / devices = different data. No sync.
