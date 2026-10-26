# Tool setup and preflight

An exploratory session only counts if you **actually drive the product**. Before anything else, check that you have a tool that can reach the target. You can tell by the tools available to you in this conversation — MCP tools usually appear with the server name in them (e.g. `mcp__playwright__browser_navigate`, or just `browser_navigate`).

## Which tool for which target

| Target | Default tool | How to recognize it's connected | Fallback |
|---|---|---|---|
| **Web app** (default) | Playwright MCP | `browser_navigate`, `browser_snapshot`, `browser_click`… | Another browser-automation MCP or a browser extension the agent already has (e.g. Claude in Chrome). Never source code alone. |
| **REST / GraphQL / SOAP API** | Shell with `curl` (+ `jq`) | You can run shell commands | An HTTP/API MCP (e.g. Postman's MCP server, an OpenAPI-based MCP, or a custom one — see api-techniques.md) |
| **Mobile (Android / iOS)** | A mobile automation MCP (e.g. `mobile-mcp`, Appium-based MCPs) | Tools to list devices, tap, swipe, read the screen | — |
| **Desktop (Windows / macOS / Linux)** | A desktop automation MCP (e.g. Windows-MCP) or the agent's computer-use capability | Tools to click, type, take screenshots of the desktop | — |
| **CLI tool** | Shell | You can run shell commands | — |

The MCP names above are examples, not endorsements — the ecosystem changes fast. Have the user check the project's page and pick one their team trusts. A database MCP (read-only user) is a great *complement* for any target: it lets you follow the data to where it's stored.

## When the tool is missing — stop and warn

Don't simulate the session, and don't "explore" by reading the code. Show a warning like this (in the user's language), then stop and wait:

> ⚠️ **Playwright MCP not connected.** This skill explores web apps by actually driving a browser, and I don't have browser tools in this session.
>
> To connect it in Claude Code:
> ```bash
> claude mcp add playwright -- npx -y @playwright/mcp@latest
> ```
> Or add it to the project's `.mcp.json` (works for any MCP-compatible agent):
> ```json
> { "mcpServers": { "playwright": { "command": "npx", "args": ["-y", "@playwright/mcp@latest"] } } }
> ```
> Then restart the session and ask again. Meanwhile I can draft the charter and a list of test ideas for this area — want that?

For non-web targets, adapt it: say which kind of tool is needed (mobile automation MCP, desktop automation MCP…), why (you need to tap/click/read the real app), and that once it's connected you'll run the session. Don't invent install commands for tools you're not sure about — point to the tool's own documentation instead.

For an API with no shell and no HTTP MCP, the warning is the same idea: you need a way to send HTTP requests.

The only things allowed without the tool are clearly labeled **preparation**: a charter, test ideas, a risk list. Never present them as results.

## Before starting — quick checks

1. **Target is up.** Open the URL / call a health or simple GET endpoint / list devices. A page that loads with HTTP 5xx or network errors in the console (e.g. `ERR_NAME_NOT_RESOLVED`, `fetch failed` to its backend) is **not** up. If it's down and local with a documented start command, try starting it once yourself (SKILL.md, step 2); if that fails, ask the user to start it. If a dependency is what's failing (a paused cloud database, an unreachable auth provider), report the host and error as an environment problem and ask the user to fix it — don't debug their infrastructure unless asked.
2. **Where are you?** Note the environment (URL, build/version if visible, device/OS) — it goes in the report header.
3. **Access.** Do you need a login or token? Use what the user gave or the project's own test users. If there are none, ask once.
4. **Session start time.** `date "+%Y-%m-%d %H:%M %s"`.
