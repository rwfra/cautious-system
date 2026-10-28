# Project Pulse — Implementation Plan

## 1. Overview

**Goal:** Build Project Pulse, a small static dashboard for Mona's team (from `.github/project-pulse-brief.md`). It shows which projects are active, who owns each one, its status, recent activity and priority/risk. The layout uses polished project cards, status badges and readable spacing. It runs from the VS Code launch configuration **Run Project Pulse Dashboard**.

**In scope**
- Static app with no dependencies: `app/index.html` (with inline JS), `app/styles.css`, `app/project-data.json`.
- Summary cards (total, on track, at risk, blocked, high priority) and a grid of project cards.
- `.vscode/launch.json` that serves `app/` over HTTP and opens `index.html`, not a directory listing.
- Loading, empty and error states.
- An accessible layout that works on phones and desktops.

**Out of scope**
- Build tools, frameworks, npm packages, a backend or API, authentication, data editing, persistence, routing, automated test harnesses.
- Changes to `.github/`, `.devcontainer/`, `.vscode/tasks.json`, `scripts/`, `README.md`.
- Git staging, commits and pushes (the learner does these).

## 2. File assignments

| File | Purpose | Owner |
|---|---|---|
| `docs/project-pulse-plan.md` | This plan | Planner |
| `app/index.html` | Semantic page structure, exact title "Project Pulse", `<link href="styles.css">`, and an **inline `<script>`** (no separate JS file) that loads `project-data.json` and renders the cards | **Coder** (Designer gives markup and class guidance) |
| `app/styles.css` | All visual styling: `.dashboard`, `.project-card`, badges, layout, responsive and focus styles | **Designer** |
| `app/project-data.json` | Data with a top-level `projects` array | **Coder** |
| `.vscode/launch.json` | **Run Project Pulse Dashboard** config: strict JSON, `python3 -m http.server 5500` from `${workspaceFolder}/app`, opens `http://localhost:%s/index.html` | **Coder** |

**Decision: keep the JS inline in `app/index.html`.** The Step 3 check looks for `project-card`, `status`, `recentActivity`, `priority` and `project-data.json` inside `index.html`. A separate `app.js` would hide those strings from the check and add a file the brief doesn't ask for.

### Shared contract (decided in Phase 1, used by everyone)

**Data schema (`app/project-data.json`)**
```json
{
  "lastUpdated": "2026-10-28",
  "projects": [
    {
      "id": "pulse-api",
      "name": "string (required)",
      "owner": "string (required)",
      "status": "on-track | at-risk | blocked | completed (required)",
      "recentActivity": "string (required)",
      "priority": "high | medium | low (required)",
      "summary": "string (optional, contributor-friendly)",
      "progress": "integer 0–100 (optional)",
      "dueDate": "YYYY-MM-DD (optional)"
    }
  ]
}
```
Sample data: 5–6 projects that together use every `status` and `priority` value, plus at least one project with no optional fields.

**DOM and CSS contract**
- `<body>` → `<header class="dashboard-header">` (h1 "Project Pulse", last-updated text), `<main class="dashboard">`.
- `<section class="summary" aria-labelledby="summary-heading">` → `.summary-card` items.
- `<section aria-labelledby="projects-heading">` → `<ul id="project-list" class="project-grid">` → `<li><article class="project-card">`.
- Inside each card: `h3.project-name`, `.project-owner`, `.status-badge.status--{on-track|at-risk|blocked|completed}`, `.priority-badge.priority--{high|medium|low}`, `.project-activity`, optional `.project-summary`, `.progress` (native `<progress>` with a label), `.project-due`.
- State container: `<div id="status-message" class="state-message" role="status" aria-live="polite">` with modifiers `.state--loading`, `.state--empty`, `.state--error`.

## 3. Designer responsibilities

**Owns:** `app/styles.css`. **Advises on:** the class and element structure in `app/index.html` (the contract above) and badge labels and wording.

- **Look:** a polished dashboard rather than a plain page. Light, neutral background; white cards with `border-radius` (about 12px) and a soft `box-shadow`; a clear type scale (system font stack). Define colors and spacing as CSS custom properties in `:root`.
- **Information hierarchy:** title, then summary counts, then project cards. Inside a card: name and badges first, then owner, then recent activity and summary, then progress and due date.
- **Status and priority treatment:** color-coded badges that also carry a text label (color is never the only signal). Optionally, high-priority cards get a left accent border.
- **Layout:** CSS Grid using `repeat(auto-fill, minmax(280px, 1fr))`. One column at ≤480px; summary cards wrap. No horizontal scrolling at 320px.
- **Accessibility:**
  - Text contrast ≥4.5:1, including badge text on badge backgrounds.
  - Visible `:focus-visible` outline.
  - Honor `prefers-reduced-motion` (hover lift and transitions only).
  - Headings in order (h1 → h2 → h3).
  - Landmarks (`header`, `main`).
  - Distinct styling for the state messages.
- **Deliverable:** before styling starts, a short design note (palette, badge colors, breakpoints) sent to the Orchestrator, then the finished `app/styles.css`.

## 4. Coder responsibilities

**Owns:** `app/project-data.json`, `app/index.html` (markup and inline JS), `.vscode/launch.json`.

- **Data:** write the sample JSON so it matches the schema exactly (field names are case-sensitive: `recentActivity`).
- **Markup:** semantic HTML that follows the DOM contract, `<title>Project Pulse</title>`, `<h1>Project Pulse</h1>`, `lang="en"`, the viewport meta tag, and `<link rel="stylesheet" href="styles.css">`.
- **Loading and rendering (inline script):**
  - Call `fetch('project-data.json')` and check `response.ok`.
  - Confirm `Array.isArray(data.projects)`.
  - Render each project with `document.createElement` and `textContent` (no `innerHTML` with data, to avoid injection).
  - Compute the summary counts.
  - Map unknown `status` or `priority` values to a neutral "unknown" badge instead of crashing.
  - Skip missing optional fields.
  - Treat a missing required field as "—" and log `console.warn`.
- **States:**
  - Loading message while the fetch runs.
  - Empty state when `projects` is `[]`.
  - Error state with a plain message when the fetch fails, a non-2xx response comes back, or the JSON is invalid. On `location.protocol === 'file:'`, add a hint telling the user to run **Run Project Pulse Dashboard**.
- **Launch config (`.vscode/launch.json`):** strict JSON with no comments or trailing commas.
  ```json
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
  ```
  `node-terminal` uses VS Code's built-in JS debugger, so the Python extension isn't needed. Python 3 is already in the universal devcontainer image.

## 5. Dependencies

**Task dependencies**
1. Shared contract (schema + DOM/class names) → needed by the JSON data, the HTML/JS rendering and the CSS selectors.
2. Designer's design direction → needed by `app/styles.css` and by badge wording and class modifiers in `app/index.html`.
3. `app/project-data.json` (sample values) → needed to check that rendering covers every status and priority.
4. `app/index.html` + `app/styles.css` + `app/project-data.json` → needed by end-to-end validation through `.vscode/launch.json`.

**Tooling and runtime dependencies**
- `fetch()` of a local JSON file fails over `file://` (CORS/origin rules), so the page **must be served over HTTP**. The launch config does this with `python3 -m http.server 5500` and `cwd` set to `app/`.
- Opening `/index.html` explicitly avoids the server's directory listing.
- In Codespaces, port 5500 is auto-forwarded and `openExternally` opens the forwarded URL.
- Requires `python3` on PATH and VS Code's built-in JS debugger (`node-terminal`). No npm packages.

## 6. Parallel work decisions (phases)

| Phase | Task | Agent | File scope | Mode |
|---|---|---|---|---|
| 1 | Approve the shared contract in §2 (schema, IDs and classes) | Orchestrator, with Designer and Coder review | none (this plan) | Sequential: everything else depends on it |
| 2a | Design direction + full stylesheet | Designer | `app/styles.css` | **Parallel** |
| 2b | Sample data | Coder | `app/project-data.json` | **Parallel** |
| 2c | Launch config | Coder | `.vscode/launch.json` | **Parallel** |
| 3 | Markup + inline rendering JS + states | Coder | `app/index.html` | Sequential: needs Phase 1 classes, 2a modifier names and 2b sample values |
| 4 | Cross-file review and fixes | Designer reviews; fixes go to the file's owner, one at a time | owners' files only | Sequential |
| 5 | Validation and launch run | Orchestrator | none (read-only) | Sequential |

**Why Phase 2 can run in parallel:** the three tasks write to three different files, and each depends only on the Phase 1 contract, not on the others' output. 2b and 2c are both Coder tasks but touch different files, so they can run concurrently or back-to-back.

**Why the other phases are sequential:**
- `index.html` connects data and styles, so it has to follow the agreed names.
- The review can change files owned by either agent, so it must not overlap with writing.
- No two concurrent tasks ever touch the same file.

## 7. Validation expectations

**Static checks**
- [ ] `python3 -m json.tool app/project-data.json` exits 0, and the file has a top-level `projects` array.
- [ ] Every project has `name`, `owner`, `status`, `recentActivity`, `priority`.
- [ ] `python3 -m json.tool .vscode/launch.json` exits 0, and the file contains `Run Project Pulse Dashboard`, `python3 -m http.server 5500`, `${workspaceFolder}/app` and `http://localhost:%s/index.html`.
- [ ] `app/index.html` contains `Project Pulse`, `styles.css`, `project-data.json`, `project-card`, `status`, `recentActivity`, `priority`.
- [ ] `app/styles.css` contains `.dashboard`, `.project-card`, `border-radius`, `box-shadow` and at least one `@media` query.
- [ ] Cross-file consistency:
  - every class or ID that the JS or HTML creates has a CSS rule (and the reverse, with no orphan selectors for core components);
  - every JSON field the renderer reads exists in the schema;
  - every `status--*` and `priority--*` value in the JSON has a CSS modifier.
- [ ] `bash scripts/validate-exercise.sh` shows no new failures (learner files are expected to be untracked until committed).

**Runtime checks**
- [ ] **Run Project Pulse Dashboard** starts the server and opens `/index.html`, and the dashboard appears, not a directory listing.
- [ ] DevTools console shows no errors, and the Network tab shows `project-data.json` returning 200.
- [ ] Number of cards equals `projects.length`. Summary counts match the data. Every project shows name, owner, status, recent activity and priority.
- [ ] States:
  - temporarily renaming the JSON shows the error state;
  - `{"projects": []}` shows the empty state;
  - invalid JSON shows the error state;
  - opening via `file://` shows the error plus the launch hint.

  Restore the original data afterward.
- [ ] Responsive at 320px, 768px and 1280px: no horizontal scroll; the grid has 1 column on phones and several on desktop.

**Accessibility checks**
- [ ] Text and badges reach ≥4.5:1 contrast (DevTools or Lighthouse).
- [ ] Badges show text labels, not just color.
- [ ] One h1; headings in order; `header` and `main` landmarks; `lang="en"`.
- [ ] Tab order is logical and the focus outline is visible.
- [ ] The state region is announced (`role="status"`).
- [ ] Lighthouse Accessibility score ≥ 90.

## 8. Risks and open questions

- **Port 5500 already in use** (for example by Live Server): the server fails to start. Stop the other process; keep 5500 because the exercise uses that fixed port.
- **Codespaces browser opening:** `openExternally` depends on port forwarding. If the browser doesn't open, use the Ports panel and add `/index.html`.
- **serverReadyAction pattern:** relies on Python's "Serving HTTP on … port N" output. If the output is buffered and the pattern never matches, the server still runs; open the page manually.
- **Inline JS vs a separate file:** inline was chosen so the Step 3 keyword checks pass. If the team later prefers `app/app.js`, the checks may fail.
- **Open questions:**
  - Should we keep the 4-value status set and 3-value priority set?
  - Are `summary`, `progress` and `dueDate` wanted, or only the five required fields?
  - Should a light/dark theme be included? (Default: light only, out of scope.)
