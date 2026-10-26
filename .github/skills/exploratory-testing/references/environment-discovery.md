# Environment discovery

This skill is installed on its own; the product under test lives somewhere else. Typically the user either:
- **adds the product's project folder to the workspace** (in Claude Code: starts in that folder, or uses `/add-dir <path>` / `--add-dir`; in VS Code: a multi-root workspace), or
- **just gives a URL / app / API address**, with no code at all.

Your job here is to work out, in a few minutes and without interrogating the user, the answers to four questions.

## 1. Where is the target?

In this order:
1. **The request.** A URL, endpoint, app name or folder the user named wins over anything you find.
2. **Workspace folders.** Check the current directory and any additional directories in the workspace. Ignore folders that are only skills or agent configuration (a folder whose content is `skills/*/SKILL.md`, `.claude/`, `.github/` and docs is a skills collection, not a product).
3. **Several candidates?** Pick the one that matches the request (the name of the screen/flow/endpoint appears in it) and say which one you picked. Ask only if it's genuinely ambiguous.
4. **Nothing found and no URL?** Ask one question: "What should I explore — a URL, or the project folder (you can add it with `/add-dir`)?"

## 2. What kind of target, and how is it reached?

Read only the files that answer this — don't scan the whole repo.

| Signal in the project | Likely target | Where to find how it's reached |
|---|---|---|
| `package.json` with a web framework (React, Next, Vue, Angular, Svelte…), `index.html`, `vite.config.*` | Web | `scripts.dev`/`start`, ports in the framework config, `.env.example` |
| E2E config: `playwright.config.*`, `cypress.config.*`, `wdio.conf.*` | Web (often with a ready base URL) | `baseURL` / `baseUrl`, test users in fixtures or support files |
| `openapi.*` / `swagger.*`, GraphQL schema, API framework (Express, Nest, FastAPI, Spring, .NET Web API, Rails API…), Postman collection, `.http` files | API | Server port, base path, auth scheme in the spec, collection variables |
| `docker-compose.*`, `Dockerfile`, `Makefile`, `Procfile` | Any | Exposed ports, service names, start commands |
| `android/`, `ios/`, `app.json` (Expo), `pubspec.yaml` (Flutter), `*.xcodeproj`, `build.gradle` with `applicationId` | Mobile | App/bundle ID, how to build/install, emulator or device |
| `*.csproj` with WPF/WinForms/MAUI, Electron, Tauri, Qt | Desktop | Executable or how to launch it |
| `bin` in `package.json`, `cmd/`, `console_scripts`, a CLI framework | CLI | The command and its `--help` |
| README / CONTRIBUTING "Running locally" sections | Any | Usually the fastest answer |

Environment variable files: read `.env.example` / `.env.sample` for names and defaults. Don't read real `.env` files unless you need a value to connect; if you do, never copy secrets into notes or the report.

If the target is reached through a deployed environment (staging URL in the README, a CI config), confirm with the user before using it — it may be shared (see Safety in SKILL.md).

## 3. What does "correct" mean here? (oracles)

Look for, in this order, and stop when you have enough for the charter:
- Requirements, user stories, acceptance criteria, business rules (`docs/`, `requirements/`, `specs/`, `*.feature`, ADRs).
- Contracts: OpenAPI/GraphQL schemas, JSON schemas, Postman collections.
- The existing automated tests for that area — they show what the team thinks the behavior is, and what's **not** covered (a good place to explore).
- Recent changes in that area (`git log --oneline -15 -- <path>`), which are where risk usually is.

No documented oracle is normal. Use the product itself, comparable products and user expectations (see heuristics.md), and record "no written requirements found for this area" as an (I) note — it also changes how you classify findings: without a source, most surprises become risks, improvements or questions rather than defects.

## 4. How do you get in?

- **Credentials:** what the user gave → test users defined by the project for testing (seed files, fixtures, E2E support files, README) → ask once. Never guess, and never use real people's accounts.
- **Data state:** the charter may need a state that doesn't exist (an out-of-stock product, an expired subscription). Check whether the project has seeds or an admin area to create it; if not, record the gap as a risk or question.
- **Tools:** whether you can actually drive the target is the next step — [tool-setup.md](tool-setup.md).

## Summarize before exploring

Tell the user what you understood in 2–4 lines and carry on without waiting, unless something is a guess that would waste the session if wrong:

> Target: web app from `~/projects/shop` (Next.js), running at `http://localhost:3000` (from `playwright.config.ts`). Oracles: `docs/rules.md` and the cart tests in `e2e/cart.spec.ts`. Login: test user from `e2e/fixtures/users.ts`. Starting the session on the cart.

These facts go in the report header (Environment) and, when relevant, as (I) notes.
