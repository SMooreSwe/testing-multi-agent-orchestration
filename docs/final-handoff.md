# Project Pulse: final handoff

Mona's **Project Pulse** dashboard is a small static app. It loads project data from JSON and shows each project as a card with its name, owner, status badge, priority, progress, due date, summary and recent activity. It has no external dependencies.

## Final Project Pulse result

- **Outcome:** a working static Project Pulse dashboard that shows 6 sample projects as cards. Each card shows the name, owner, status badge, priority, progress, due date, summary and recent activity.
- **Dashboard features:** a status summary strip with counts per status, an "Overdue" marker on past-due projects, and loading, empty and error states.
- **Styling:** responsive grid layout with accessible styling: visible keyboard focus, forced-colors and reduced-motion support, and ARIA labels and live regions in the markup.
- **App files created:** `app/index.html`, `app/styles.css` and `app/project-data.json`, plus `.vscode/launch.json` with the **"Run Project Pulse Dashboard"** launch configuration.
- **Validation:** all automated checks passed (checks 1–9 below). The manual browser checks (check 10) are left for the learner.

## Agent contributions

The agents are defined in `.github/agents/` and described in `docs/agent-team.md`.

| Agent | Contribution |
| --- | --- |
| **Orchestrator** | Split the request into tasks, gave each specialist a non-overlapping file scope, ran the Designer and Coder phases in parallel, sequenced the integration validation, and reported the result. It did not write any code. |
| **Planner** | Wrote `docs/project-pulse-plan.md`: file assignments, the shared Designer ⇄ Coder contract (section 5: class names, data attributes, JSON schema, cleanup rules), phases, edge cases and validation expectations. |
| **Designer** | Wrote `app/styles.css`: dashboard layout, project cards, status badges, priority tags, progress bar, overdue and loading/empty/error state styling, responsive grid, focus styles, forced-colors and reduced-motion support. |
| **Coder** | Wrote `app/index.html` (markup and inline fetch/render script), `app/project-data.json` (6 sample projects) and `.vscode/launch.json` (launch configuration). Also ran the validation below and wrote this handoff. |

## Files created

These files were created for this project.

| Path | Purpose |
| --- | --- |
| `app/index.html` | Page titled "Project Pulse". Links `styles.css`, fetches `project-data.json`, and renders `.project-card` elements with `textContent` (no `innerHTML`). Handles the loading, empty and error states. |
| `app/styles.css` | All styling. Includes the `.dashboard` and `.project-card` hooks and every class in the contract. |
| `app/project-data.json` | Top-level `projects` array with 6 projects. Together they cover every status and every priority. |
| `.vscode/launch.json` | The **"Run Project Pulse Dashboard"** launch configuration. |

## How to run

- **Run and Debug panel:** open Run and Debug, choose **Run Project Pulse Dashboard** and start it. This runs `python3 -m http.server 5500` with `cwd` set to `${workspaceFolder}/app`. When the server is ready, `serverReadyAction` opens `http://localhost:5500/index.html`.
- **Terminal:** run `cd app && python3 -m http.server 5500`, then open http://localhost:5500/index.html.

Opening `index.html` directly over `file://` shows an error message on purpose, because browsers block `fetch()` of local files.

## Dashboard validation

Run on 2026-11-11 against the committed files (`bb4d048`).

| # | Check | Command / method | Result |
| --- | --- | --- | --- |
| 1 | `app/project-data.json` is strict JSON | `python3 -m json.tool app/project-data.json` | ✅ Pass |
| 2 | Data schema: `projects` array of 6; every project has `name`, `owner`, `status`, `recentActivity`, `priority`, `progress`, `dueDate`, `summary`; all 5 statuses and all 3 priorities appear; progress is an integer from 0 to 100; dates are `YYYY-MM-DD`; ids are unique | Read-only `python3` check | ✅ Pass |
| 3 | `.vscode/launch.json` is strict JSON with name exactly "Run Project Pulse Dashboard", `cwd` `${workspaceFolder}/app`, command `python3 -m http.server 5500`, and `uriFormat` `http://localhost:%s/index.html` | `python3 -m json.tool` + `python3` field check | ✅ Pass |
| 4 | `app/index.html`: `<title>Project Pulse</title>`, `<h1>`, `styles.css` link, fetches `project-data.json` with `cache: "no-store"`, renders `.project-card`, contains `status`/`recentActivity`/`priority`, no `innerHTML` or `<style>`, uses every contract class (5.1) | Static `python3` check | ✅ Pass |
| 5 | `app/styles.css`: `.dashboard`, `.project-card`, `border-radius`, `box-shadow`; styles all 29 contract classes; covers every `data-status` (including `unknown`), `data-priority` (including `unknown`) and `data-state`; `data-overdue`; `var(--progress)`; `[hidden] { display: none !important; }`; responsive `auto-fill`/`minmax` grid; `:focus-visible`; `prefers-reduced-motion`; `overflow-wrap: anywhere`; no external imports | Static `python3` check | ✅ Pass |
| 6 | HTTP serving: `index.html`, `styles.css` and `project-data.json` each return 200 with the correct content type; served HTML contains "Project Pulse" and served JSON is valid; an unknown file returns 404 | `python3 -m http.server 5500` + `curl` (server stopped afterwards) | ✅ Pass |
| 7 | Rendering: 6 cards in JSON order; summary counts (not-started 1, on-track 2, at-risk 1, blocked 1, completed 1) match the data; `CI Pipeline Migration` shows "Overdue"; `--progress` is set; the "Last updated" subtitle is shown | Inline script run in Node 22 with a minimal DOM shim and real `fetch` against the server | ✅ Pass |
| 8 | States: empty → "No projects to show yet."; invalid JSON, missing `projects`, HTTP 404, network failure and `file://` → `data-state="error"` with `role="alert"` and a specific message | Same Node harness with fake responses | ✅ Pass |
| 9 | Edge cases: `"At Risk"` → `at-risk`; unknown priority → `unknown`/"Not set"; `<script>` text in a name stays literal; duplicate or missing ids get unique values; progress 150 → 100%; non-numeric progress → "Not reported" and the bar is hidden; missing date → "No due date" | Same Node harness | ✅ Pass |
| 10 | Manual browser checks (plan section 10, items 13, 18, 21 and 22): launch config opens `/index.html`, responsive breakpoints, contrast, keyboard focus, 200% zoom | Need a real browser | ⏳ Not run. For the learner. |

## Final handoff notes

### Known issues

No blocking defects were found. No app files were changed during validation. Minor observations:

- The `accessibility-audit` summary is 164 characters. The plan aims for at most 160. This looks like the intended long-text sample, and the CSS wraps or clamps it.
- Overdue status uses the browser's local date (plan open question 4). The result changes depending on when the dashboard is viewed. Today, only `CI Pipeline Migration` is overdue.
- `node-terminal` with `serverReadyAction` has not been tested in a real VS Code Run and Debug session here (plan open question 2).
- Port 5500 is fixed. If another process such as Live Server is using it, the server won't start. Stop that process first.
- In the empty state, the summary strip still shows zero counts. The plan allows this.

### Next steps

1. Start **Run Project Pulse Dashboard** from Run and Debug and confirm that `/index.html` opens, not a directory listing.
2. In DevTools, check the layout at 320px, 375px, 768px and 1280px or wider, badge contrast, Tab focus, and 200% zoom.
3. Optional: decide on the open questions in plan section 11 (dark mode, sorting/filtering, how "today" is defined for overdue).

### Git

Git is controlled by the learner. No agent staged, committed or pushed anything. This file (`docs/final-handoff.md`) is the only new file from this step and is left uncommitted for the learner to review.
