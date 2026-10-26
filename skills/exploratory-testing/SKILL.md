---
name: exploratory-testing
description: Conducts real exploratory testing (Session-Based Test Management) of web apps, APIs, mobile, or desktop apps, driving the product with MCP tools (Playwright MCP for web; HTTP tools for APIs). Use when asked for exploratory testing, an exploratory session, bug hunting, to "test freely," SBTM, a charter or session report, or to investigate a screen, flow, or endpoint for defects and risks. First discovers the target, access method, and requirements, then verifies the necessary tools are connected. Produces a session report with a charter, information and risk notes, evidence-backed defects, questions, coverage gaps, and next charters. Not for scripted test cases/Gherkin, page-object selector mapping, or documenting an already-found bug (use bug-report-writer).
---

# Exploratory Testing

Do what a good exploratory tester does: **learn the product, design tests and run them at the same time**, letting each result decide the next action. There's no script. The value is in noticing what nobody wrote in a requirement.

Freedom is not randomness. What separates exploratory from ad-hoc testing is a purpose (the charter), reasoning about what to try next, and recording what you learned.

**What to read:** [references/environment-discovery.md](references/environment-discovery.md) and [references/tool-setup.md](references/tool-setup.md) always, before touching the product. Then, by target: [web-techniques.md](references/web-techniques.md) for web, [api-techniques.md](references/api-techniques.md) for APIs. [heuristics.md](references/heuristics.md) when you run out of ideas or the charter names a heuristic. [session-report.md](references/session-report.md) when writing the report, and [labels.md](references/labels.md) when it isn't in English.

## Language
Before replying, look for a `.qa-skills.yaml` file — first in the project root, then in the home directory (`~/.qa-skills.yaml`). Use `language` for everything you say to the user and `report_language` (defaults to `language`) for the session report. An explicit request in the conversation overrides the file: one that clearly targets only the report changes just the report; any other changes both. With neither, use the language of the user's message. A missing file is normal — don't mention it. Evidence (UI text, messages, payloads, logs) is never translated.

## The mindset

- **Curiosity over coverage.** You're not walking through screens; you're asking the product questions. "What happens if…?" drives the session.
- **Learn → design → execute → observe → learn.** If something looks odd, pull that thread — that's usually where the bug is.
- **Distrust the happy path.** It almost always works. Problems live at the edges, in interruptions, combinations, odd data and half-stated rules.
- **You are the oracle — and you say why.** There's no written expected result. Judge against the project's requirements and docs, the API contract, the rest of the product, what a user would expect and common sense — and always state *which* of these the behavior contradicts. When the explored area has no requirements of its own, look at the rules of the areas that depend on it: they show what it must guarantee.
- **Follow the data to the other side.** Data entered in one place is used in others: later steps of a flow, another role's view, an admin panel, an email, after a reload, in the API response. A strange value one screen accepts often becomes real damage only at the other end. Go and check.
- **Separate the interface from the server.** Validation in the UI doesn't protect the data. When the UI blocks something, ask whether the server blocks it too.
- **Observe more than the screen.** Console, network requests, status codes, URL, what persists after reload, what changes in another tab or for another user.
- **Learn by hand, sweep in batches.** Do the first experiments step by step to understand the target. Then, to try many data variations, run them in one batch that returns only the result — faster and much cheaper in context.
- **The charter guides, it doesn't imprison.** If something important shows up outside it, note it and decide consciously whether to detour.
- **Take notes while testing**, not at the end — fresh notes keep details that vanish from memory.

## Honesty rules
The report is only useful if every line in it really happened.
- Only report what you **observed** through the tools in this session. Reading code or docs gives context and oracles; it is not a test result. If you inferred something without executing it, label it as a risk or question, never as a defect.
- Never simulate a session. If the tools to reach the product aren't available, say so (see step 2) instead of describing what "would probably" happen.
- Before citing a screenshot or log as evidence, look at it and confirm it shows what you claim.
- Write what you did *not* cover. A short, honest "Not covered" list is worth more than implied completeness.

## How a session runs

### 1. Understand the environment
You arrive knowing nothing about the product, and the folder where this skill lives is never the product. Before exploring, find out **where the product is, what kind it is, how to reach it and what "correct" means** — following [references/environment-discovery.md](references/environment-discovery.md). In short:
- **Locate the target.** A URL or app the user named; a project folder they added to the workspace; or both. Look in the workspace folders, not only the current directory.
- **Read just enough** of that project to learn how it runs and where it is reached (base URL, ports, API base path, app ID), the test users it defines for testing, and its oracles (requirements, stories, OpenAPI/GraphQL schemas, docs).
- **No project folder is fine** — with only a URL, explore black-box and say in the report that there were no requirements to compare against.
- Ask only for what you couldn't find and can't proceed without — usually one question (the URL, or which folder is the product). Summarize what you found in 2–4 lines before moving on, so the user can correct a wrong assumption.

### 2. Preflight — can you reach the product?
With the target type known (web, API, mobile, desktop, CLI), follow [references/tool-setup.md](references/tool-setup.md):
- **Web is the default and uses the Playwright MCP.** If its browser tools (`browser_navigate`, `browser_snapshot`, …) aren't available, **stop and show the warning** from tool-setup.md with the setup command. Don't fall back to guessing from source code.
- **Non-web targets need an MCP or tool that can drive them** (a mobile automation MCP, a desktop automation MCP…). Same rule: warn, explain what's needed, stop.
- **APIs** work with any HTTP client you can run — a shell with `curl`, or an HTTP/API MCP.

Then confirm the target is up (load the URL, call a health endpoint) — "up" means it answers without 5xx or network errors in the console, not just that the port responds. If it's down:
- **Local target with a documented start command** (README, `package.json` scripts, `docker compose up`, `npx <server>`): say which command you'll run, then try it once in the background and wait for it to answer. Never run setup wizards, migrations or anything that asks for credentials or changes a shared environment.
- **If that fails, or there's no documented command, or the target isn't local:** stop, tell the user what you tried and what you saw, show the command the project documents (if any) and ask them to start it.
- **Up but broken by a dependency** (5xx because a database, auth or third-party service is unreachable): it's an environment problem, not a defect. Report what failed (host, error) and ask the user to fix it — don't debug their infrastructure.

### 3. Safety
- **Credentials:** use what the user gives, or test users the project already defines for testing. Never guess or reuse real people's credentials, and never copy secrets from `.env` files into notes or the report.
- **Environment:** local and test environments are fair game. On anything shared, staging-with-real-users or production, **ask before any action that changes state** (creating, paying, deleting, sending emails) — reading is fine. Bypassing the UI to hit the server directly, auth/authorization probing and high-volume batches are only for environments the user explicitly authorized. Never do load/stress or anything that could take the service down.
- **Test data:** make what you create easy to recognize (a prefix like `qa-explore-`, a fixed ID range) and keep a running list of every record you create. Never use real personal data. If the data store is volatile (in-memory server, container without a volume), say so in the report — restarting it is the cleanup.

### 4. Charter
If the user brought a charter, use it. If they brought only a target ("explore the checkout"), write one — **Explore** <target> **With** <heuristic / resource / focus> **To discover** <information> — and go on without waiting for approval, unless the request is really vague. With no target at all, pick the area with most risk (recent changes, money, permissions, data that flows to other places) and say why.

Timebox: you act much faster than a person, so "10 minutes" signals **depth** (a short, focused session) more than a stopwatch. Without guidance, think of a 20–30 human-minute session. Record start and end time (`date "+%Y-%m-%d %H:%M %s"` — the `%s` makes the difference easy) and report the real wall-clock duration plus the number of experiments (e.g. "6 min (≈35 experiments)").

### 5. Explore
Navigate, try, vary, interrupt, repeat with different data. Keep a draft of short notes, tagged:
- **(I) Information** — what you did, learned or noticed, including what worked well and what led you to the next experiment.
- **(R) Risk** — not a proven defect (yet), but something that could harm users or the business; state the consequence.

Capture evidence when something looks like a defect (screenshot, request/response, console error), numbered `01-…`, `02-…` so the report can cite them. When something surprising happens, reproduce it once more before calling it a defect — and note whether it reproduced.

When you hit the timebox, stop — a session is closed time. Close the browser/app session.

### 6. Classify what you found
If the project defines what counts as a defect, follow its definition. Otherwise:
- **Defect** — reproducible, and contradicts an explicit source (requirement, contract/schema, docs, the product's own messages or behavior elsewhere) **or** is an unambiguous failure: crash, unhandled error/5xx, data loss or corruption, wrong calculation, security or privacy exposure. Write it so someone can reproduce it: where, what you did (with the exact data), what happened, what was expected and *by which source*, evidence.
- **Improvement** — looks bad by common sense, consistency or user expectation, but nothing forbids it. Record as an (R) note starting with "Improvement:" and say the impact.
- **Question** — business-rule doubts only someone from product can answer ("Should it be possible to…?"). Ambiguity isn't a defect, it's a question. Check the docs first; if the docs are silent, the default is "no restriction", so only ask when the doubt has real impact.

Findings outside the charter go into the notes, and if they are defects, into the defect list prefixed with "(Off-charter)".

### 7. Report
Write the session report following [references/session-report.md](references/session-report.md) and save it to `exploratory-sessions/<YYYY-MM-DD>-<target-slug>/session.md`, with evidence files next to it — inside the product's project folder when there is one (if it already has a folder or template for session reports, use that), never inside the skills folder. With no project folder (URL only), save it in the current working directory and say where. If you create a new `exploratory-sessions/` folder in a git repository, tell the user it isn't ignored and ask whether it should be committed or added to `.gitignore` — reports may contain project data. Clean up temporary tool folders (e.g. `.playwright-mcp/`) after moving the evidence you need.

If the user later answers the questions, update the report: add an (I) note with the answers, mark each question as answered, and reclassify the findings.

### 8. Hand back
Tell the user where the report is and give a short summary: the charter, how many defects / risks / questions, and the 2–3 most relevant findings. Then:
- If test data was created, list it and **ask** whether to delete it — never delete on your own; similar-looking data may belong to someone else.
- Offer to turn the defects into tickets with the `bug-report-writer` skill (if installed).
- Suggest 1–3 next charters for areas that deserved more time.
