# Web techniques with the Playwright MCP

Tricks that worked in real sessions. Use them when they help and adapt when needed. Tool names may vary slightly between Playwright MCP versions (e.g. `browser_run_code` vs `browser_run_code_unsafe`) — use the one you have.

## Reading the page cheaply

- `browser_snapshot` gives the accessibility tree — great for learning a new screen. On pages with big lists it gets huge; a `browser_evaluate` that summarizes only what matters (card texts, `aria-disabled`, `data-*` attributes) saves context and often reveals things the screen doesn't show.
- Read text from `body`, not `main`: not every page has `<main>`, and a locator waiting for it can hang until timeout.

## Running many experiments at once

One click per call is great to *learn* a screen. Once you understand it, testing variations one by one (limits, formats, hostile data, filter combinations, quantities) wastes context. Run a batch with the code-execution tool: each case starts from a clean `goto`, applies the variation, acts and returns only what matters.

```js
async (page) => {
  const URL = 'http://localhost:3000/<route>';
  const base = { /* valid reference values */ };
  const cases = [ { /* only what changes in this case */ } ];
  const out = [];
  page.on('dialog', d => { out.push({ DIALOG: d.message() }); d.dismiss(); });
  for (const c0 of cases) {
    const c = { ...base, ...c0 };
    await page.goto(URL);
    // apply c: page.fill / selectOption / click, using selectors you already found
    // act: submit, search, continue
    await page.waitForTimeout(1200);
    const body = await page.locator('body').innerText();
    out.push({ c: c0, url: page.url(), excerpt: body.slice(-300) });
  }
  return out;
}
```

Tips:
- If a case creates a record, use a unique identifier per case. A case that "should fail" may be saved, and then the next one fails for duplication — which confuses the reading.
- Trim the returned text to the part that matters so the answer doesn't grow for nothing.
- **Guard every case.** A case you expected to be rejected may be accepted and move the app to the next step — then the next `fill` waits for a field that no longer exists and the whole batch hangs until the tool times out (minutes). Set a short timeout (`page.setDefaultTimeout(4000)`), check after each case that you're still where you expect (URL, step heading), and record + stop at the first deviation instead of continuing blind. Prefer a fresh `goto` per case in multi-step flows.
- Explore by hand first; the batch sweeps variations once you know what to look for. If something surprises you, go back to step-by-step and pull the thread.

## Reading what the screen doesn't show

- Form attributes reveal client-side rules (`required`, `min`, `max`, `step`, `maxLength`, `pattern`, select options):
  `[...document.querySelectorAll('input,select,textarea')].map(e => ({name: e.name, type: e.type, req: e.required, min: e.min, max: e.max, step: e.step, maxl: e.maxLength, pattern: e.pattern}))`.
  Compare with what the app really accepts — gaps between the two often hide defects (e.g. `step=0.01`, but `100.999` is accepted and silently rounded).
- Before inventing URL parameters, collect the real links on the page (`a[href]`) to learn the names and values the app uses. Inventing parameters on purpose is a good contrarian approach — just know when you're doing it.
- Watch for new console entries in the MCP responses and check `browser_console_messages` / `browser_network_requests`: unhandled exceptions and 4xx/5xx that the UI swallows show up there.

## Testing behind the UI (client vs server)

Only on environments the user authorized (see SKILL.md, step 3).

1. Capture the real request of a valid action (save, book, pay):
   `page.on('request', r => r.method() !== 'GET' && log.push({ url: r.url(), body: r.postData() }))`.
   Frameworks with server functions/RPC use opaque URLs and serialized bodies — copy the format and change only the values.
2. Resend it with `fetch` inside `page.evaluate` (it carries the session cookies) with data the UI wouldn't allow: an option that isn't in the select, empty field, negative number, invalid format, a resource that's already taken, a quantity above the limit, another user's ID.
3. In logged-in areas, repeat after logout (and as a different user/role) to check authorization.

## Signals that look like tool errors but are information

- A `click` that times out with "element is not enabled" right after another click means the app disabled the button during submission — double-click protection. Note it as (I) and confirm how many records were created.
- A `goto` redirected to the login page means the session dropped. Log in again before concluding anything about the screen.
- A `browser_resize` to a phone width is a cheap way to find layout and hidden-button issues.

## Evidence

- `browser_take_screenshot`, or `page.screenshot({ path: '.playwright-mcp/01-description.png', fullPage: true })` inside a code batch. Number the files (01, 02…) and cite the number in the defect.
- The Playwright MCP writes its files (screenshots, snapshots, console logs) under `.playwright-mcp/` in the project. At the end, move the evidence into the session folder and delete `.playwright-mcp/` so nothing untracked is left behind.
- Look at each screenshot (`Read`) before citing it, to confirm it shows what you claim.
