# Project Pulse final handoff

Mona's Project Pulse dashboard is built, validated, and pushed to `main`. GitHub Copilot CLI in a Codespace coordinated four custom agents to do the work.

## Agent team

- **Orchestrator** split the work into phases, gave each specialist an explicit file scope, sent review findings to the right owner, and checked the combined result. It wrote no code.
- **Planner** researched the repository and wrote `docs/project-pulse-plan.md`: file ownership, shared data and markup contract, dependencies, parallel work decisions, edge cases, and validation expectations.
- **Designer** owned all visual and accessibility decisions and wrote `app/styles.css`.
- **Coder** wrote `app/index.html`, `app/project-data.json`, and `.vscode/launch.json`, and ran the validation commands.

Agent definitions live in `.github/agents/`. See `docs/agent-team.md` for target models and responsibilities.

## What was built

- `app/index.html`: the dashboard page, titled exactly "Project Pulse". It links `styles.css`, fetches `project-data.json`, and renders one `.project-card` per project. Each card shows the name, owner, status, recentActivity, priority, and summary when there is one.
  - Data is inserted with `textContent` and `createElement` only, never `innerHTML`.
  - Status and priority values are cleaned up. Unknown values get a neutral "unknown" badge that still shows the original text.
  - Loading, empty ("No projects yet"), and error messages appear in a live region. Opening the page from disk (`file://`) shows a message that points to the launch configuration.
- `app/styles.css`: a polished, responsive dashboard.
  - `.dashboard` and `.project-card` selectors, rounded corners (`border-radius`), and soft shadows (`box-shadow`).
  - A grid that shows one column on phones and several on desktop.
  - Color-coded status badges with text labels. A colored left border shows priority, widest for high.
  - Visible focus outlines, support for reduced motion, and text contrast planned to meet WCAG AA.
- `app/project-data.json`: a top-level `"projects"` array of 6 projects. Each has `name`, `owner`, `status`, `recentActivity`, and `priority`. Together they cover every status (on-track, at-risk, blocked, completed) and every priority (high, medium, low), and one project has a very long name to test wrapping.
- `.vscode/launch.json`: strict JSON with one configuration named "Run Project Pulse Dashboard".
  - Runs `python3 -m http.server 5500` with `cwd` set to `${workspaceFolder}/app`.
  - Its `serverReadyAction` opens `http://localhost:%s/index.html`, so the dashboard opens instead of a list of files.

## Final validation results

| Check | Result |
|---|---|
| `app/project-data.json` and `.vscode/launch.json` are strict JSON (`python3 -m json.tool`) | Pass |
| 6 projects, each with all 5 required fields as non-blank text; every status and priority is used | Pass |
| The launch configuration name, `cwd`, `command`, and `uriFormat` match exactly; no comments | Pass |
| `app/index.html` contains the title, `styles.css`, `project-data.json`, `project-card`, `status`, `recentActivity`, and `priority`; no `innerHTML` | Pass |
| `app/styles.css` contains `.dashboard`, `.project-card`, `border-radius`, `box-shadow`, `@media`, and a grid layout | Pass |
| Every class the page uses has a matching style in `app/styles.css` | Pass |
| The page's script parses without errors (Node `vm.Script`) | Pass |
| Local server on port 5500: `/index.html`, `/styles.css`, and `/project-data.json` return 200; `/` serves the dashboard, not a file list; the server was stopped afterwards | Pass |
| `scripts/validate-exercise.sh` | 2 failures, neither in the app (see below) |

`scripts/validate-exercise.sh` has 2 failures:

- **Learner files are tracked.** Expected: the exercise script deliberately fails once learner files are committed.
- **README doesn't mention "Project Pulse".** The template's `README.md` doesn't include the phrase. This is a template check, not an app problem.

### Not yet validated

No headless browser is available in the Codespace, so these were not checked automatically:

- the 6 rendered cards
- the layout at 320px, 768px, and 1280px
- keyboard focus
- reduced motion
- color contrast, measured with a contrast checker

## How to run

1. Open **Run and Debug** in VS Code.
2. Choose **Run Project Pulse Dashboard** and start it.
3. The browser opens `http://localhost:5500/index.html` (in Codespaces, the forwarded port URL) and shows 6 project cards.

If port 5500 is already in use, stop the other process and run it again.

## Next steps for the handoff

- Run the manual checks in **Not yet validated** above.
- Decide the open questions in `docs/project-pulse-plan.md`: whether the status list is right, whether `recentActivity` should include a date, and whether the header should show summary counts.
- To change projects, edit `app/project-data.json`. Keep the 5 required fields, use the allowed status and priority values, and no code changes are needed.
- Git is controlled by the learner. Every commit and push was made only on an explicit request.
