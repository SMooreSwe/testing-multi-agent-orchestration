# Project Pulse dashboard: implementation plan

## 1. Summary

Mona's **Project Pulse** dashboard is a small, frontend-only static app. It shows Mona's projects as polished cards. Each card shows the project's name, owner, status badge, priority, progress, due date, short summary and recent activity.

- **Data:** `app/project-data.json` has a top-level `projects` array. Each project includes `name`, `owner`, `status`, `recentActivity` and `priority`, which the brief requires. It also includes `progress`, `dueDate` and `summary`.
- **Markup + logic:** `app/index.html` has the exact title "Project Pulse" and links to `styles.css`. Its inline `<script>` loads `project-data.json` with `fetch()` and builds one `.project-card` per project. It also handles the loading, empty and error states.
- **Styling:** `app/styles.css` gives a polished, responsive, accessible look. It must include the `.dashboard` and `.project-card` selectors and use `border-radius` and `box-shadow`.
- **Run/preview:** `.vscode/launch.json` adds a launch configuration named **"Run Project Pulse Dashboard"**. It starts `python3 -m http.server 5500` with `cwd` set to `${workspaceFolder}/app`. Its `serverReadyAction` then opens `http://localhost:%s/index.html`. A server is needed because browsers block `fetch()` of local JSON over `file://`. Opening `index.html` directly avoids a server directory listing.

The work is split between two agents with no shared files: **Designer** (CSS) and **Coder** (HTML/JS, data, launch config). The shared contract in section 5 fixes the class names, data attributes and JSON schema up front. That lets the two implementation phases run in **parallel**. The **Orchestrator** runs integration and validation afterwards, one step at a time.

Constraints from the repository's own conventions:
- Agents must not stage, commit, or push. The learner controls git.
- `.vscode/launch.json` must be strict JSON with no comments.
- The devcontainer has no Python VS Code extension. It does have `python3` and VS Code's built-in JavaScript debugger. So the launch config uses `"type": "node-terminal"` to run the shell command, not `debugpy`.
- Do not modify the existing `.vscode/tasks.json`.

## 2. File assignments

| File | Owning agent | Purpose |
| --- | --- | --- |
| `app/index.html` | **Coder** | Page structure, "Project Pulse" title, `<link>` to `styles.css`, inline `<script>` that fetches `project-data.json`, checks and cleans up values, renders `.project-card` elements, and handles loading/empty/error states. |
| `app/styles.css` | **Designer** | All visual styling: `.dashboard` layout, `.project-card`, status badges, priority treatment, progress bar, state messages, responsive grid, focus styles, contrast, reduced motion. |
| `app/project-data.json` | **Coder** | Sample dataset with a top-level `projects` key that follows the schema in section 5.4. |
| `.vscode/launch.json` | **Coder** | Strict-JSON launch config "Run Project Pulse Dashboard" that serves `app/` over HTTP on port 5500 and opens `index.html`. |

No file has two owners. These assignments match the ones given in the request, so no changes needed justifying.
- The Designer must **not** edit `app/index.html`. If a markup change is needed for styling, the Designer asks the Orchestrator, who sends it to the Coder.
- The Coder must **not** edit `app/styles.css`. The Coder must not add `<style>` blocks or presentational inline styles to `app/index.html`. The one exception is the `--progress` CSS custom property defined in section 5.
- No other files are created or modified. That includes `.vscode/tasks.json`, `README.md` and the `.github/` files.

## 3. Designer responsibilities (`app/styles.css`)

Based on `.github/agents/designer.agent.md`:

1. **Polished dashboard, not a bare page.** The first view should clearly read as a "Project Pulse" dashboard:
   - a styled header area
   - a summary strip of status counts
   - a grid of cards on a soft neutral page background
2. **Required CSS hooks.** The stylesheet must contain the literal selectors `.dashboard` and `.project-card`. It must style every class in the contract in section 5.1, using those exact names.
3. **Project cards.** Use `border-radius`, `box-shadow`, padding and a clear internal order:
   - project name (most prominent)
   - status badge
   - owner
   - summary
   - meta (priority, due date)
   - progress bar
   - recent activity

   An optional hover/focus-within lift should have a `prefers-reduced-motion` fallback.
4. **Status badges.** Style `.status-badge` using `[data-status="…"]` for every value in section 5.2, including `unknown`. Each status gets its own color. The badge always shows a text label, so color is never the only signal.
5. **Priority treatment.** Style `.priority-tag[data-priority="…"]` for `high`, `medium`, `low` and `unknown`. Optionally add a left border accent on `.project-card[data-priority="high"]`. High priority must stand out without clashing with the status colors. One option is a solid tag for high and outlined tags for medium/low.
6. **Progress bar.** `.progress-bar` is the track and `.progress-bar__fill` is the fill. The fill width comes from the custom property set on the card, `width: var(--progress, 0%)`. Style `.project-card[data-status="completed"]` and `[data-status="blocked"]` fills to match their status.
7. **Overdue treatment.** Style `.project-card__due[data-overdue="true"]`, for example with a warning color plus the "Overdue" text the Coder renders.
8. **State messages.** Style `.dashboard__state` for each `data-state` value: `loading`, `empty` and `error`. The error state must be clearly visible (a different color with a border or icon, plus text). The `hidden` attribute must still hide the element (`[hidden] { display: none; }` must not be overridden).
9. **Responsive layout.**
   - Use a CSS Grid on `.project-grid`, e.g. `repeat(auto-fill, minmax(min(100%, 18rem), 1fr))`.
   - 1 column at about 360px wide, 2 or more on tablet, 3 or more on desktop.
   - No horizontal scroll at 320px.
   - The header and summary strip wrap cleanly.
10. **Long text.** Use `overflow-wrap: anywhere` on names, owners, summaries and activity. Optionally clamp the summary to about 3 lines with `-webkit-line-clamp`, but the full text must stay in the DOM.
11. **Accessibility and contrast.**
    - Text contrast of at least 4.5:1, including badge text on badge backgrounds.
    - Visible `:focus-visible` outlines.
    - Readable base font size (≥ 16px) and line-height (≥ 1.4).
    - Honor `prefers-reduced-motion`.
    - Style `.visually-hidden` as the standard screen-reader-only utility.
12. **No external dependencies.** Use a system font stack and no CDN fonts or frameworks, so the app works offline in a Codespace.
13. **Report back** with design decisions, tradeoffs (for example the color palette and line clamping), files touched (only `app/styles.css`) and validation recommendations.

## 4. Coder responsibilities

### 4.1 `app/project-data.json`
- Strict JSON (no comments, no trailing commas), UTF-8, 2-space indent.
- Top-level object with a `projects` array, plus an optional `lastUpdated` field (see section 5.4).
- **6 sample projects** that between them cover every allowed `status` (`not-started`, `on-track`, `at-risk`, `blocked`, `completed`) and every `priority` (`high`, `medium`, `low`). Include:
  - at least one project with a deliberately long name/summary to test wrapping
  - at least one `dueDate` in the past on a project that is not `completed`, to show the overdue state
- Every project includes all fields in section 5.4, so the automated keyword checks pass and the sample data is complete.

### 4.2 `app/index.html`
- `<!DOCTYPE html>`, `<html lang="en">`, `<meta charset="utf-8">` and a viewport meta tag.
- `<title>Project Pulse</title>` and `<h1>` text **"Project Pulse"**.
- `<link rel="stylesheet" href="styles.css">` (relative path).
- Static markup skeleton exactly as described in section 5.3.
- An inline `<script>` placed at the end of `<body>`, or with `defer`-style behavior, that:
  1. Sets the state to `loading`.
  2. Calls `fetch("project-data.json", { cache: "no-store" })` with a relative URL.
  3. Treats non-OK HTTP responses, JSON parse errors, a missing `projects` key and a `projects` value that is not an array as **errors**. Each case gets a clear, specific message.
  4. If the page is on `location.protocol === "file:"`, shows an error that explains the dashboard must be opened through the **Run Project Pulse Dashboard** launch configuration (served over HTTP).
  5. Cleans up each project (section 5.5): trims strings and turns `status`/`priority` into lowercase, hyphenated slugs. Unknown values map to `unknown`. `progress` is limited to 0–100. `dueDate` is checked.
  6. Renders cards **in JSON order** using `document.createElement` and `textContent`, **never** `innerHTML` with data values. This prevents HTML injection from the data.
  7. Fills in the summary counts per status.
  8. Sets the state to `empty` if `projects` is an empty array. Otherwise it hides the state element and shows the grid.
  9. Logs details with `console.error` on failure. The user-facing message stays short.
- The script must contain the field names `status`, `recentActivity` and `priority` literally. They will be there naturally because the script reads those properties.
- No external scripts or libraries.

### 4.3 `.vscode/launch.json`
Strict JSON with no comments. Exact intended content:

    {
      "version": "0.2.0",
      "configurations": [
        {
          "name": "Run Project Pulse Dashboard",
          "type": "node-terminal",
          "request": "launch",
          "command": "python3 -m http.server 5500",
          "cwd": "${workspaceFolder}/app",
          "serverReadyAction": {
            "action": "openExternally",
            "pattern": "Serving HTTP on .* port ([0-9]+)",
            "uriFormat": "http://localhost:%s/index.html"
          }
        }
      ]
    }

Why this shape:
- `node-terminal` is built into VS Code through the JavaScript debugger. It runs any shell command in a debug terminal and supports `serverReadyAction`. The devcontainer does not install the Python extension, so `debugpy` might not be available.
- `cwd` set to `${workspaceFolder}/app` serves the `app/` folder as the web root.
- Python's `http.server` prints `Serving HTTP on 0.0.0.0 port 5500 (http://0.0.0.0:5500/) ...`. The pattern captures the port, and `uriFormat` opens `/index.html` directly, so the learner never sees a directory listing.
- In Codespaces, opening a `localhost` URL is automatically port-forwarded.

### 4.4 Coder report
List the files touched, the commands run for validation (section 10) and any remaining risks.

## 5. Shared contract (Designer ⇄ Coder)

This contract is fixed before Phase 1. Neither agent changes a name in it without going through the Orchestrator.

### 5.1 Class names (BEM-style)

| Class | Element | Notes |
| --- | --- | --- |
| `dashboard` | `<main>` | Root layout container (required hook). |
| `dashboard__header` | `<header>` | Title area. |
| `dashboard__title` | `<h1>` | Text: "Project Pulse". |
| `dashboard__subtitle` | `<p>` | e.g. "Mona's projects at a glance" plus "Last updated" when present. |
| `dashboard__summary` | `<ul>` | Status count strip. |
| `summary-stat` | `<li>` | One per status; has `data-status`. |
| `summary-stat__count` | `<span>` | Number. |
| `summary-stat__label` | `<span>` | Status label text. |
| `dashboard__state` | `<div>` | Loading/empty/error message; has `data-state`. |
| `project-grid` | `<ul>` | Card grid. |
| `project-grid__item` | `<li>` | Wraps one card. |
| `project-card` | `<article>` | One project (required hook); has `data-status` and `data-priority`. |
| `project-card__header` | `<header>` | Name + status badge. |
| `project-card__name` | `<h2>` | Project name. |
| `status-badge` | `<span>` | Has `data-status`; text label always present. |
| `project-card__owner` | `<p>` | Owner line. |
| `project-card__label` | `<span>` | Small label text ("Owner", "Recent activity"). |
| `project-card__summary` | `<p>` | Summary text. |
| `project-card__meta` | `<dl>` | Priority + due date. |
| `project-card__meta-item` | `<div>` | Wraps one `<dt>`/`<dd>` pair. |
| `priority-tag` | `<span>` | Has `data-priority`. |
| `project-card__due` | `<time>` | Has `datetime`; `data-overdue="true"` when overdue. |
| `project-card__progress` | `<div>` | Progress section. |
| `progress-bar` | `<div>` | Track; `role="progressbar"`. |
| `progress-bar__fill` | `<div>` | Fill; width comes from `--progress`. |
| `progress-bar__value` | `<span>` | Text such as "65%" or "Not reported". |
| `project-card__activity` | `<p>` | Recent activity line. |
| `visually-hidden` | any | Screen-reader-only text. |

### 5.2 Data attributes and allowed values

| Attribute | On | Allowed values | Display label (Coder renders) |
| --- | --- | --- | --- |
| `data-status` | `.project-card`, `.status-badge`, `.summary-stat` | `not-started`, `on-track`, `at-risk`, `blocked`, `completed`, `unknown` | Not started, On track, At risk, Blocked, Completed, Unknown |
| `data-priority` | `.project-card`, `.priority-tag` | `high`, `medium`, `low`, `unknown` | High, Medium, Low, Not set |
| `data-state` | `.dashboard__state` | `loading`, `empty`, `error` | (message text) |
| `data-overdue` | `.project-card__due` | `true`, or absent | adds visible text "Overdue" |
| `style="--progress: N%"` | `.project-card` | `0%`–`100%`, or absent when unknown | — |

The `--progress` custom property is the only inline style the Coder may write. The Designer reads it in `.progress-bar__fill { width: var(--progress, 0%); }`. When progress is unknown, the Coder adds the `hidden` attribute to `.progress-bar` and the value text reads "Not reported".

### 5.3 HTML structure

Static skeleton in `index.html`:

    <body>
      <main class="dashboard" aria-labelledby="dashboard-title">
        <header class="dashboard__header">
          <h1 id="dashboard-title" class="dashboard__title">Project Pulse</h1>
          <p class="dashboard__subtitle">…</p>
        </header>
        <ul class="dashboard__summary" aria-label="Projects by status"></ul>
        <div class="dashboard__state" data-state="loading" role="status" aria-live="polite">Loading projects…</div>
        <ul class="project-grid" aria-label="Projects"></ul>
      </main>
      <script>…</script>
    </body>

The error state switches the element to `role="alert"`, or puts the message in a child that has `role="alert"`.

Each rendered card, created with the DOM API:

    <li class="project-grid__item">
      <article class="project-card" data-status="at-risk" data-priority="high"
               style="--progress: 40%" aria-labelledby="project-<id>-name">
        <header class="project-card__header">
          <h2 class="project-card__name" id="project-<id>-name">…</h2>
          <span class="status-badge" data-status="at-risk"><span class="visually-hidden">Status: </span>At risk</span>
        </header>
        <p class="project-card__owner"><span class="project-card__label">Owner</span> …</p>
        <p class="project-card__summary">…</p>
        <dl class="project-card__meta">
          <div class="project-card__meta-item"><dt>Priority</dt><dd><span class="priority-tag" data-priority="high">High</span></dd></div>
          <div class="project-card__meta-item"><dt>Due</dt><dd><time class="project-card__due" datetime="2026-12-01">Dec 1, 2026</time></dd></div>
        </dl>
        <div class="project-card__progress">
          <div class="progress-bar" role="progressbar" aria-valuemin="0" aria-valuemax="100"
               aria-valuenow="40" aria-label="Progress for …">
            <div class="progress-bar__fill"></div>
          </div>
          <span class="progress-bar__value">40%</span>
        </div>
        <p class="project-card__activity"><span class="project-card__label">Recent activity</span> …</p>
      </article>
    </li>

### 5.4 JSON schema (`app/project-data.json`)

    {
      "lastUpdated": "2026-11-11",
      "projects": [
        {
          "id": "pulse-api",
          "name": "Pulse API",
          "owner": "Mona",
          "status": "on-track",
          "priority": "high",
          "progress": 65,
          "dueDate": "2026-12-15",
          "summary": "Short contributor-friendly description (aim for ≤ 160 chars).",
          "recentActivity": "Merged rate-limiting PR; reviewing auth changes."
        }
      ]
    }

| Field | Type | Required in sample data | Allowed values / format |
| --- | --- | --- | --- |
| `lastUpdated` (top level) | string | optional | `YYYY-MM-DD` |
| `projects` (top level) | array | **yes** | array of project objects (may be empty) |
| `id` | string | yes | unique, lowercase, `a-z0-9-`; used for element ids. If missing, the Coder generates `project-<index>`. |
| `name` | string | **yes** | non-empty |
| `owner` | string | **yes** | non-empty |
| `status` | string | **yes** | `not-started`, `on-track`, `at-risk`, `blocked`, `completed` |
| `priority` | string | **yes** | `high`, `medium`, `low` |
| `progress` | integer | yes | 0–100 |
| `dueDate` | string | yes | ISO date `YYYY-MM-DD` |
| `summary` | string | yes | plain text, no HTML |
| `recentActivity` | string | **yes** | plain text, no HTML |

Fields marked **yes** in bold are required by `.github/project-pulse-brief.md` and the automated checks in `.github/workflows/3-step.yml`.

### 5.5 Cleanup and fallback rules (Coder implements; Designer can rely on them)
- `status`/`priority` values are trimmed and lowercased, and spaces/underscores become `-`. So `"On Track"` becomes `on-track`. Any value not in the allowed list becomes `unknown`.
- Missing or blank `name` → "Untitled project".
- Missing `owner` → "Unassigned".
- Missing `summary` → "No summary provided."
- Missing `recentActivity` → "No recent activity."
- `progress` that is not a number → treated as unknown. Numbers are rounded and limited to 0–100.
- Missing or invalid `dueDate` → "No due date", rendered as a `<span class="project-card__due">` with no `datetime`.
- A project is overdue when `dueDate` is before today's local date and `status` is not `completed`.
- Dates are shown in a fixed format, `en-US` with `{ month: "short", day: "numeric", year: "numeric" }`. Parse them as local dates to avoid off-by-one timezone shifts. Do **not** use `new Date("YYYY-MM-DD")`, which treats the date as UTC.

## 6. Ordered implementation steps / phases

**Phase 0 – Plan and contract (done by the Planner).** Save this document as `docs/project-pulse-plan.md`. The Orchestrator confirms the contract in section 5 with both specialists.

**Phase 1 – Parallel implementation**
- **1A (Designer):** Create `app/styles.css` according to sections 3 and 5.
- **1B (Coder):** Create `app/project-data.json` according to sections 4.1 and 5.4, and `.vscode/launch.json` according to section 4.3.
- **1C (Coder):** Create `app/index.html` according to sections 4.2 and 5.3–5.5.

1B and 1C can be one Coder task or two separate Coder tasks. Both are in the Coder's file scope and they don't share any files. If 1C runs before 1B finishes, the Coder tests 1C against the schema in section 5.4.

**Phase 2 – Integration validation (sequential, after all of Phase 1).** The Orchestrator delegates running the checks in section 10 to the Coder, because the Coder has execute tools. The Orchestrator itself only reads the results and the files.

**Phase 3 – Fix-ups (sequential, only if needed).** Each problem goes to the owner of the file involved:
- CSS → Designer
- HTML/JS, JSON, launch → Coder

If a fix changes the contract, the Orchestrator updates both sides one after the other. Re-run the relevant section 10 checks afterwards.

**Phase 4 – Learner checkpoint.** The learner runs **Run Project Pulse Dashboard** from Run and Debug, then handles git through Copilot CLI. The final handoff (`docs/final-handoff.md`) is a later exercise step and is outside this plan's file scope.

## 7. Dependencies between steps

| Step | Depends on | Kind of dependency |
| --- | --- | --- |
| 1A styles.css | Phase 0 contract (section 5.1–5.2) | Interface only (class/attribute names) |
| 1B project-data.json | Phase 0 schema (section 5.4) | Interface only |
| 1B launch.json | Nothing in app files | None; only needs the `app/` folder path |
| 1C index.html | Phase 0 contract (section 5.1–5.5) | Interface only. At runtime it needs `project-data.json` to exist (needed for Phase 2, not for writing the code). |
| Phase 2 validation | 1A, 1B, 1C all complete | Data + file dependencies |
| Phase 3 fix-ups | Phase 2 findings | Sequential |
| Phase 4 learner run | Phase 2/3 passing | Sequential |

## 8. Parallel work decisions

- **Parallel: 1A (Designer) ∥ 1B and 1C (Coder).**
  - **File scope:** the steps touch completely separate files (`styles.css` vs `index.html`/`project-data.json`/`launch.json`), so they can't overwrite each other's edits.
  - **Data dependency:** the CSS only depends on class and attribute *names*, which section 5 fixes up front. It doesn't depend on the HTML file's content.
- **Parallel: 1B ∥ 1C.** These are different files with the same owner. `index.html` depends on the JSON *schema*, which is fixed, not on the data file existing at authoring time. `launch.json` is independent of both.
- **Sequential: Phase 2 after Phase 1.** Validation needs every file to exist. The real HTTP load needs `index.html`, `project-data.json` and `styles.css` together.
- **Sequential: Phase 3 after Phase 2.** Fixes depend on validation results. If a fix touches the contract, run the Designer and Coder changes one after the other so the class names stay consistent.
- **Not parallel:** never give two agents the same file in the same phase.

## 9. Edge cases

| Case | Expected behavior | Owner |
| --- | --- | --- |
| `projects` is `[]` | `.dashboard__state[data-state="empty"]` shows "No projects to show yet." The grid is empty; summary counts are all 0 or hidden. | Coder (logic), Designer (style) |
| `projects` missing or not an array | Error state: "Project data is malformed: expected a 'projects' list." | Coder |
| Invalid JSON | Error state: "Could not read project data (invalid JSON)." | Coder |
| Fetch fails (404, server stopped, network) | Error state with the HTTP status if there is one, e.g. "Could not load project-data.json (HTTP 404)." | Coder |
| Opened via `file://` | Error state explaining that the dashboard must be run with **Run Project Pulse Dashboard** (served over HTTP). | Coder |
| Missing fields on a project | Fallback text from section 5.5; the card still renders. | Coder |
| Unknown status/priority value | Rendered with `data-status="unknown"` / `data-priority="unknown"` and the label "Unknown" / "Not set", with a neutral style. | Coder + Designer |
| Mixed-case/spaced status (`"At Risk"`) | Converted to `at-risk`. | Coder |
| `progress` out of range, string or missing | Limited to 0–100. If not a number, the bar is hidden and the text reads "Not reported". | Coder |
| Invalid or past `dueDate` | Invalid → "No due date". Past and not completed → `data-overdue="true"` plus the text "Overdue". | Coder + Designer |
| Long name/owner/summary/activity, long unbroken words or URLs | Wraps with `overflow-wrap: anywhere`; no card overflow. The optional summary clamp keeps the full text in the DOM. | Designer |
| HTML-like text in data (`<script>`) | Shown as literal text because the code uses `textContent`. | Coder |
| Duplicate or missing `id` | Generate `project-<index>` so the element ids stay unique. | Coder |
| Small screens (320–375px) | Single column, no horizontal scroll, header/summary wrap, touch-friendly spacing. | Designer |
| Large screens | Content max-width (e.g. ~1200px) centered; grid grows to 3–4 columns. | Designer |
| Port 5500 already in use | The server fails to start. See the open questions; the learner stops the other process or the plan changes the port. | Coder (document in report) |
| Browser cache of old JSON | `cache: "no-store"` on the fetch. | Coder |
| Many projects (e.g. 50+) | Grid scrolls normally; no performance concern at this size. | — |

## 10. Validation expectations

The Coder runs these in Phase 2 and the Orchestrator reviews the output. Every check must pass.

**JSON validity**
1. `python3 -m json.tool app/project-data.json >/dev/null` exits 0.
2. `python3 -m json.tool .vscode/launch.json >/dev/null` exits 0. This also proves there are no comments.

**Data schema** (a quick `python3 -c` read-only check)
3. The top-level `projects` key exists and is a list of 6 projects.
4. Every project has `name`, `owner`, `status`, `recentActivity`, `priority`, `progress`, `dueDate` and `summary`.
5. Every `status` value is one of the 5 allowed values, and all 5 appear.
6. Every `priority` value is `high`, `medium` or `low`, and all 3 appear.
7. Each `progress` is an integer from 0 to 100, and each `dueDate` matches `YYYY-MM-DD`.

**Launch config**
8. `.vscode/launch.json` contains the name "Run Project Pulse Dashboard", the command `python3 -m http.server 5500`, the `cwd` `${workspaceFolder}/app`, and `serverReadyAction.uriFormat` `http://localhost:%s/index.html`.

**Static content checks** (same keywords the automated workflow checks)
9. `app/index.html` contains "Project Pulse", `styles.css`, `project-data.json`, `project-card`, `status`, `recentActivity` and `priority`.
10. `app/styles.css` contains `.dashboard`, `.project-card`, `border-radius` and `box-shadow`.
11. `app/styles.css` has a selector for every class in section 5.1 and every `data-status`/`data-priority` value in section 5.2, including `unknown`.

**HTTP serving**
12. From `app/`, start `python3 -m http.server 5500` in the background. Then check:
    - `curl -sf http://localhost:5500/index.html` returns HTML containing "Project Pulse".
    - `curl -sf http://localhost:5500/project-data.json` returns valid JSON.
    - `curl -sf http://localhost:5500/styles.css` returns 200.
    - Stop the server afterwards.

**Rendering** (manual, via the launch config)
13. **Run Project Pulse Dashboard** opens `/index.html`, not a directory listing.
14. 6 `.project-card` elements render inside `.dashboard`. Each shows the name, owner, status badge, priority, progress, due date, summary and recent activity.
15. Status counts in `.dashboard__summary` match the data.
16. The overdue project shows "Overdue".

**Error state**
17. Opening `app/index.html` via `file://`, or blocking `project-data.json` in DevTools, shows a visible `.dashboard__state[data-state="error"]` message. No project files need to be edited for this test.

**Responsive**
18. In the DevTools device toolbar:
    - at 320px and 375px: 1 column, no horizontal scrollbar
    - at about 768px: 2 columns
    - at about 1280px or wider: 3 or more columns

**Accessibility basics**
19. One `<h1>`, with an `<h2>` per card. `<html lang="en">` is set.
20. The progress bar has `role="progressbar"` and `aria-valuenow` (or is hidden when unknown). The state element has `role="status"`/`aria-live`, and errors use `role="alert"`.
21. Status and priority always show text, not only color. Badge and body text pass 4.5:1 contrast (e.g. in the DevTools contrast checker or Lighthouse accessibility).
22. Tabbing shows visible focus. Layout is usable at 200% zoom.

**Scope**
23. `git status` shows only the four assigned files plus `docs/project-pulse-plan.md` as new or changed. Nothing is staged.

Note: `scripts/validate-exercise.sh` checks the exercise template itself. It includes a check that learner answer files are **not** tracked, so it isn't a pass/fail signal for this dashboard work.

## 11. Open questions

1. **Port 5500:** Is a fixed port acceptable? The exercise prompt asks for `python3 -m http.server 5500`. If the port might be in use (e.g. by Live Server), should we document the conflict or choose another port?
2. **Launch type:** `node-terminal` relies on VS Code's built-in JavaScript debugger, and `serverReadyAction` matches the server's terminal output. This should work in Codespaces, but it hasn't been tested in this environment. If it fails, a fallback is a `preLaunchTask`. That would mean editing `.vscode/tasks.json`, which is outside the current file scope and would need Orchestrator approval.
3. **Card order:** JSON order (current plan) or sorted by priority/status/due date? Should there be filter or sort controls? These are not in scope now.
4. **"Today" for overdue:** The browser's local date is used, so the overdue result depends on when the dashboard is viewed. Is that acceptable, or should it be measured against `lastUpdated`?
5. **Dark mode:** Should the Designer support `prefers-color-scheme: dark`, or ship light-only?
6. **Sample data content:** Should the projects be fictional (Mona/Octocat-themed), or does Mona have real project names to use?
7. **Summary length:** Hard limit in the data (≤ 160 characters), or just visual clamping in CSS?
