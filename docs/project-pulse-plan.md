# Project Pulse Implementation Plan

## Summary

Mona's team needs **Project Pulse**: a small static dashboard that shows contributors, at a glance:

- which projects are active
- who owns each project
- each project's status
- recent activity
- priority
- a short summary

It's made of three files:

- `app/index.html`
- `app/styles.css`
- `app/project-data.json`

It also needs a VS Code launch configuration named **Run Project Pulse Dashboard** in `.vscode/launch.json`. That configuration starts a local web server from `app/` and opens `http://localhost:5500/index.html`, so the dashboard appears instead of a list of files.

Sources: `.github/project-pulse-brief.md`, `.github/steps/2-step.md`, `.github/steps/3-step.md`, `.github/workflows/3-step.yml` and `.github/agents/*.agent.md`.

## File assignments (one owner per file)

- `app/project-data.json`: **Coder**
- `app/index.html`: **Coder** (builds the page from the Designer's class list below)
- `.vscode/launch.json`: **Coder**
- `app/styles.css`: **Designer**
- `docs/project-pulse-plan.md`: **Planner** (this file)

Out of scope: no agent may edit `.github/`, `.devcontainer/`, `.vscode/tasks.json`, `scripts/` or `docs/agent-team.md`.

## Shared contract (freeze before Phase 1)

Both specialists build against this contract. Changing it means re-planning both files.

### Data schema (`app/project-data.json`)

- The file holds one object with a top-level `"projects"` array.
- Each project has these required fields, all strings:
  - `name`
  - `owner`
  - `status`
  - `recentActivity`
  - `priority`
- Optional fields: `summary` (string) and `id` (unique string).
- Allowed `status` values:
  - `on-track`
  - `at-risk`
  - `blocked`
  - `completed`
- Allowed `priority` values:
  - `high`
  - `medium`
  - `low`
- Include 5 or 6 sample projects that use every status and every priority.
- Include one project with a very long name, to test text wrapping.

### Page structure and class names (`app/index.html` → `app/styles.css`)

- `<header class="dashboard-header">` holds `<h1>Project Pulse</h1>` and a one-line subtitle.
- `<main class="dashboard">` is the root that the styles target.
- `<p class="dashboard-message" role="status" aria-live="polite">` shows the loading, empty and error messages.
  - Modifier classes: `--loading`, `--empty`, `--error`
- `<ul class="project-grid">` contains one `<li>` per project.
- Each `<li>` contains `<article class="project-card" data-status="…" data-priority="…">`.
- Inside each card:
  - `h2.project-card__name`
  - `p.project-card__owner`
  - `span.status-badge.status-badge--{status|unknown}`
  - `span.priority-badge.priority-badge--{priority|unknown}`
  - `p.project-card__activity`, labelled "Recent activity"
  - `p.project-card__summary`, left out when there is no summary

### Launch configuration (`.vscode/launch.json`)

- Configuration name: `Run Project Pulse Dashboard`
- `type`: `node-terminal` (built into VS Code, so no extra extension is needed)
- `command`: `python3 -m http.server 5500`
- `cwd`: `${workspaceFolder}/app`
- `serverReadyAction`:
  - `pattern`: `Serving HTTP on .* port ([0-9]+)`
  - `uriFormat`: `http://localhost:%s/index.html`
  - `action`: `openExternally`
- The file must be strict JSON: no comments and no trailing commas.

## Designer responsibilities

- Own `app/styles.css` only. Don't edit the HTML; send any markup changes to the Coder through the Orchestrator.
- Make the dashboard look polished, not like bare HTML. The first view must clearly read as the Project Pulse dashboard.
- Use the required selectors: `.dashboard` and `.project-card`.
- Add `border-radius` and `box-shadow` on cards, plus readable spacing.
- Use a responsive grid:
  - `grid-template-columns: repeat(auto-fill, minmax(min(100%, 280px), 1fr))`
  - one column on phones, several columns on desktop
- Status badges: give each status its own color, and always show the text label too, so meaning never depends on color alone.
- Give `--unknown` a neutral gray style.
- Priority: use a clear visual weight, for example a colored left border driven by `data-priority`, and make `high` stand out the most.
- Typography: use a system font stack, a clear heading order, and a body size of at least 16px.
- Accessibility:
  - contrast of at least 4.5:1 for normal text
  - visible `:focus-visible` outlines
  - respect `prefers-reduced-motion`
- Long text: `overflow-wrap: anywhere` on names and activity text.
- Give the loading, empty and error message styles their own look.
- Report your design decisions and trade-offs.

## Coder responsibilities

- Own `app/project-data.json`, `app/index.html` and `.vscode/launch.json`.
- `index.html` setup:
  - `lang="en"`, a viewport meta tag and `<title>Project Pulse</title>`
  - link to `styles.css`
  - an inline script that calls `fetch('project-data.json')`
- Render each project as a visible `.project-card` that shows the name, owner, status, `recentActivity`, priority and summary.
  - The words `status`, `recentActivity` and `priority` must appear literally in the HTML file, because the Step 3 check searches for them.
- Use `textContent` and `createElement` to insert data. Never use `innerHTML` with data (this prevents injected script).
- Clean up values before using them as class names:
  - trim spaces, convert to lowercase, replace spaces with `-`
  - map anything outside the allowed lists to `unknown`
  - still show the original text in the badge
- Show readable labels: `on-track` becomes "On track".
- Handle errors as described in the Edge cases section.
- Write `.vscode/launch.json` exactly as in the contract.
- Run the validation commands and report the results.

## Implementation phases (in order)

1. **Phase 0 – Confirm the contract** (Orchestrator)
   - Confirm the shared contract above and pass it word for word to both specialists.
2. **Phase 1 – Build** (in parallel)
   - **Task 1A – Coder:** `app/project-data.json`, then `app/index.html`, then `.vscode/launch.json`.
   - **Task 1B – Designer:** `app/styles.css`
3. **Phase 2 – Review** (sequential)
   - The Designer reads `index.html` and the rendered page without editing them, then lists any mismatches between the markup and the styles, plus accessibility issues.
   - Markup fixes go to the Coder. Style fixes go to the Designer.
4. **Phase 3 – Fixes** (sequential, only if Phase 2 found issues)
   - The Coder and the Designer may work in parallel here, because they edit different files.
5. **Phase 4 – Validation** (Coder runs the commands; the Orchestrator checks the results)
   - Run every check in the Validation expectations section, then report.

## Dependencies

- `index.html` depends on the data schema: it reads the field names and allowed values. Both are owned by the Coder, so there's no handoff between agents.
- `styles.css` depends on the class contract, not on the finished HTML file. That's why it can be built at the same time.
- `launch.json` depends only on the file name `index.html` and the folder `app/`. Both are fixed now.
- Phase 2 depends on Phase 1 being finished, because it needs the real files.
- Validation depends on every file existing.

## Parallel work decisions

- **Run in parallel: Task 1A (Coder) and Task 1B (Designer).**
  - The files don't overlap.
  - The only thing they share is the contract, which is fixed before they start, so neither waits for the other's output.
- **Sequential within the Coder's task:** write the data first, then the HTML, then the launch file. One agent owns all three, and the HTML reads the data.
- **Sequential: Phase 2 runs after Phase 1.** The review needs both finished files.
- **Sequential: validation runs last.** It needs the combined result.
- **Never** let two agents edit the same file in the same phase.

## Edge cases

- **Opening the page directly from disk (file://).** `fetch` fails there. Catch the error and show a clear message in `.dashboard-message--error` that says to use **Run Project Pulse Dashboard**.
- **Data can't be loaded or isn't valid JSON** (HTTP error or parse error). Show the error message, and include the reason in a developer console log.
- **`projects` is missing or isn't a list.** Treat it as an error with a clear message.
- **Empty list.** Show the message "No projects yet".
- **A field is missing or blank.** Show "Not set" (or "Unassigned" for the owner) instead of leaving a gap. Don't let it break the page.
- **Unknown status or priority.** Use the neutral `--unknown` badge style and show the original text.
- **Status or priority written as "At Risk" or " HIGH ".** Clean it up before matching.
- **Long names or activity text.** Wrap the text without breaking the card layout.
- **Small screens (320px wide).** One column, no sideways scrolling, and buttons or links large enough to tap.
- **Port 5500 already in use.** The server won't start, so you get no browser tab and an error in the terminal. Stop the other process and run again.
- **Codespaces.** `openExternally` opens the forwarded port address. Check that the dashboard loads, not a list of files.
- **Data containing script.** Inserting it with `textContent` keeps it from running.

## Validation expectations

- `python3 -m json.tool app/project-data.json` exits 0.
- `python3 -m json.tool .vscode/launch.json` exits 0, which proves the file is strict JSON.
- A short Python check confirms that every project has `name`, `owner`, `status`, `recentActivity` and `priority`.
- `grep -q 'Run Project Pulse Dashboard' .vscode/launch.json`
- `grep -q 'index.html' .vscode/launch.json`
- `grep -q '${workspaceFolder}/app' .vscode/launch.json`
- `app/index.html` contains each of these:
  - `Project Pulse`
  - `styles.css`
  - `project-data.json`
  - `project-card`
  - `status`
  - `recentActivity`
  - `priority`
- `app/styles.css` contains each of these:
  - `.dashboard`
  - `.project-card`
  - `border-radius`
  - `box-shadow`
- Serve check:
  - Run `(cd app && python3 -m http.server 5500)`.
  - `curl -sf http://localhost:5500/index.html` returns HTML containing "Project Pulse".
  - `curl -sf http://localhost:5500/project-data.json` returns the data.
  - Stop the server afterward.
- Manual check: **Run and Debug → Run Project Pulse Dashboard** opens `/index.html` and shows the cards, not a file list.
- Responsive check: test widths of 320px, 768px and 1280px. Look for no sideways scrolling and for the column count to change.
- Accessibility basics:
  - one `h1`
  - `lang` is set
  - landmarks (`header`, `main`)
  - text labels on badges
  - visible focus outline
  - the error and empty messages are announced through `aria-live`
- Don't use `scripts/validate-exercise.sh` as the pass/fail check. It deliberately fails once learner files such as these are tracked in git.

## Open questions

- Should the Designer edit `app/index.html` directly? This plan says no, to keep one owner per file. The Designer's markup changes go to the Coder instead.
- Should `index.html` include a copy of the data to fall back on when opened from disk? This plan says no, keeping one source of truth and showing an error message instead.
- Is the status list (`on-track`, `at-risk`, `blocked`, `completed`) right for Mona's team? Is "risk" separate from "priority"?
- Should `recentActivity` be plain text, or include a date (for example `lastUpdated` as an ISO date) for sorting?
- Should the header show summary counts, for example "3 on track · 1 blocked"?
- Is a fixed port 5500 acceptable, or should the plan expect it to be already in use?
