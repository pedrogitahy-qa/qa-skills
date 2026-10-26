# Session report template

Inspired by Jonathan Bach's *Session-Based Test Management* (2000). Write it in `report_language`, using the labels from [labels.md](labels.md). Notes are written in the first person, in natural language — they tell the story of the session. Drop optional sections that would be empty; mandatory ones stay (an empty Defects section says "No defects found in this session.").

```markdown
# Session Report — <Module / target>

| Start | Tester | Module / target | Environment | Duration |
|---|---|---|---|---|
| 2026-10-06 20:15 | Claude (AI) | Checkout — cart | http://localhost:3000 · build 1.4.2 | 9 min (≈40 experiments) |

## Charter
*Explore* the cart quantity field
*With* boundary values and client-vs-server checks
*To discover* whether invalid quantities can reach an order

## Notes
*(I) Information · (R) Risk*
- (I) I added one item and changed the quantity to 0; the "Remove item?" dialog appeared, which matches the cart in the mobile app.
- (I) The field has `min=1 max=10` in the HTML, so I resent the update request with quantity 50 directly to the server.
- (R) The server accepted quantity 50 and the total was recalculated. If the stock check also only happens in the UI, a customer could order more units than exist.
- (R) Improvement: the error for quantity 11 says "Invalid value" without telling the limit; users will guess.

## Defects
1. **Cart accepts quantity above the maximum when the request bypasses the UI.**
   Where: `PATCH /api/cart/items/{id}` (cart screen). Data: item `qa-explore-01`, quantity `50`.
   What happened: 200 OK, cart total R$ 2.500,00. Expected: rejection — requirement RN-12 limits 10 units per item.
   Reproduced: 2/2. Evidence: `03-cart-50-units.png`, request/response in `requests.md` #4.

## Questions
- Should the 10-unit limit be per item or per order? RN-12 says "per purchase".

## Not covered
- Coupons combined with quantity changes (out of time).
- Mobile layout.

## Test data created
- Cart items with prefix `qa-explore-` (IDs 9001–9006) — cleanup pending the user's OK.

## Next charters
- Explore stock reservation with two concurrent carts to discover whether the last unit can be sold twice.
```

## Field guide

- **Start** — the session's start date and time in the local format of `report_language`.
- **Tester** — "Claude (AI)" (translated per labels.md), unless the user asks for another name. If the session was requested by someone, it can be "Claude (AI) — requested by <name>" only when the user gave the name.
- **Environment** — URL / API base URL / app version / device, exactly as observed. If you couldn't see a version, don't invent one.
- **Duration** — real wall-clock time, plus approximate experiment count.
- **Defects** — numbered, each with: where (screen path or method + endpoint), data used, what happened (exact messages), expected and **the source** of that expectation, reproducibility, evidence file names. Secrets masked. Off-charter defects start with "(Off-charter)".
- **Questions** — when answered later, keep them and append "Answered — …".
- **Not covered**, **Test data created** and **Next charters** are what make the session useful to the next person; include them whenever there's something to say.

## Optional: time split (SBTM metrics)
If the user or team tracks SBTM metrics, add an estimated split of the session: **Setup** (getting the environment ready) / **Testing** (exploring) / **Bug investigation** (reproducing and documenting). Mark it as an estimate.
