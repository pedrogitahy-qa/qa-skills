# Heuristics toolbox

Heuristics are **fallible shortcuts** for generating test ideas — not checklists or scripts. Pick one when you're out of ideas, when the charter asks for it, or when something odd shows up and you want to poke it from another angle. Drop it when it stops paying off.

## Where to look — SFDIPOT (James Bach, "San Francisco Depot")
- **Structure**: what the feature is made of (fields, components, routes, endpoints).
- **Function**: what it does; what it should do and what it does beyond that.
- **Data**: what goes in, comes out, is stored, is calculated; sizes, formats, empties.
- **Interfaces**: UI, API (network requests), console, URL, keyboard, files, integrations.
- **Platform**: browser, screen size, device, OS, multiple tabs, network conditions.
- **Operations**: how real people use it — in a hurry, distracted, coming back later.
- **Time**: dates, expiry, order of events, slowness, concurrency.

## Noticing something is wrong — oracles (FEW HICCUPPS, Michael Bolton)
A behavior is suspicious when it's inconsistent with: the product's **History**, the **Image** the company wants to project, **Comparable products**, **Claims** (docs, requirements, labels, messages), **User expectations**, the **Product itself** (another screen does it differently), its **Purpose**, **Standards**, **Explainability** (can you explain why it did that?), the real **World**.
Use this to justify a defect: "inconsistent with X because…".

## Generating variations
- **Contrarian approach / "do what you shouldn't"**: break the business rule on purpose (buy something sold out, the same seat twice, origin = destination).
- **Goldilocks**: too small, too big, just right — and exactly at the limit.
- **Zero, one, many**: no items, one item, the maximum, the maximum + 1.
- **Never and always**: what should the system never allow? What should always happen?
- **CRUD**: create, read, update, delete — and do them out of order.
- **Interruptions**: back button, reload, close and reopen the tab, double-click fast, abandon mid-flow and come back by URL, lose connection.
- **Concurrency**: two tabs/users/requests fighting for the same resource.
- **Hostile/odd data**: leading/trailing spaces, accents, emoji, only spaces, huge pasted text, special characters, upper/lower case, HTML/script as text.
- **Personas**: the hurried user, the one who gets everything wrong, the power user, the keyboard-only user, the screen-reader user.
- **Follow the money / follow the data**: does the value that went in show up the same on every following screen, in the API, after reload, for another role, in emails and receipts?
- **Hint vs reality**: compare what the UI *promises* (placeholder, format hint, `min`/`step`/`maxLength`, mask, error message) with what it really accepts.
- **Calendar edges**: yesterday, today, distant past, year 9999, month and year turn, February 29, minimum/maximum age, card expiry, time zones and DST. Check *derived* values (return date, duration, age) too — they often break before the date itself.
- **Numbers at storage limits**: 0, minimum, minimum − one decimal, extra decimals, scientific notation (`1e3`), a huge value. Applies to prices, quantities, documents, cards. Database errors leaking to the UI ("numeric field overflow") show up here.
- **Client vs server**: does the server block what the UI blocks? And without an active session? (See web-techniques.md / api-techniques.md.)
- **Accessibility quick pass**: tab through the flow, check focus visibility and labels in the snapshot, zoom to 200%.

## Structuring the charter
"**Explore** <target> **with** <resources / heuristic / technique> **to discover** <information>" (Elisabeth Hendrickson, *Explore It!*). Good charters are focused enough to guide and open enough to allow surprise.

✅ *Explore the cart quantity field with boundary values and client-vs-server checks to discover whether invalid quantities can reach an order.*
❌ *Test the cart.* (no focus) · ❌ *Check that entering 0 shows "invalid quantity".* (that's a scripted test case)
