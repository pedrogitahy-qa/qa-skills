# API exploration techniques

Exploring an API is the same mindset with a different interface: there's no screen to look at, so the **status code, headers, body and what changed in the data** are your observations.

## Tools

- **Default: shell + `curl` (+ `jq`).** Zero setup, works for REST, GraphQL and SOAP, and every request you send is a reproducible command you can paste into a defect.
- **HTTP/API MCP** (Postman's MCP server, an OpenAPI-based MCP, or a custom one) when the agent has no shell, or when the team wants guardrails: an allowlisted base URL, auth injected from environment variables, and every request logged automatically.
- Whatever the tool, **never print or save real tokens**. Read them from environment variables (`-H "Authorization: Bearer $API_TOKEN"`) and mask them in the report.

## Getting started

1. **Find the contract.** OpenAPI/Swagger (`openapi.yaml`, `/swagger.json`, `/v3/api-docs`, `/openapi.json`, `/docs`), GraphQL schema (introspection, if enabled — that it's enabled may itself be a finding in production), Postman collections, README. The contract is your main oracle.
2. **Make one valid call per main resource** and read the response carefully: fields, types, IDs, timestamps, links, headers (`Content-Type`, caching, rate-limit, CORS).
3. **Keep a request log** in the session folder (`requests.md` or `.http`) with each meaningful request and a trimmed response — it becomes evidence.

A useful base for each call:

```bash
curl -sS -X POST "$BASE_URL/orders" \
  -H "Authorization: Bearer $API_TOKEN" -H "Content-Type: application/json" \
  -d '{"productId": 1, "quantity": 2}' \
  -w '\n--- %{http_code} in %{time_total}s\n' | tee -a requests.log
```

## Batches

Once you understand an endpoint, sweep variations in one script that prints a compact table (case → status → key field / error message) instead of one call at a time:

```bash
for q in 0 1 -1 999999 1.5 '"2"' null; do
  printf '%s -> ' "$q"
  curl -sS -o /dev/null -w '%{http_code}\n' -X POST "$BASE_URL/orders" \
    -H "Authorization: Bearer $API_TOKEN" -H "Content-Type: application/json" \
    -d "{\"productId\": 1, \"quantity\": $q}"
done
```

If a case creates a record, use a unique identifier per case and record it for the cleanup list.

## Heuristics specific to APIs

- **Contract vs reality.** Required fields really required? Types enforced (`"2"` vs `2`, `null`, arrays where an object is expected)? Undocumented fields in the response? Documented status codes actually returned?
- **Status code honesty.** Validation errors as 400/422, not 500. Not found as 404. A 200 with `{"error": …}` in the body. A 500 that leaks a stack trace, SQL or internal paths.
- **Auth and authorization** (only on authorized environments): no token, expired/garbage token, a token from another user, another user's resource ID (BOLA/IDOR), an admin-only route with a normal user, changing your own role/owner/price fields in the body (mass assignment).
- **CRUD out of order.** Update or delete something already deleted; read after delete; create twice with the same unique key; PUT vs PATCH with partial bodies.
- **Idempotency and repetition.** Send the same POST twice quickly (double charge? duplicate order?). Retries with the same idempotency key, if the API supports one.
- **Concurrency.** Two requests for the same last unit of stock / the same seat at once (`&` in the shell + `wait`).
- **Lists.** Pagination edges (page 0, negative, beyond the last, huge page size), sorting and filter parameters with odd values, empty results.
- **Data at the limits.** Empty strings, whitespace, very long strings, Unicode/emoji, numbers at storage limits, dates (leap years, time zones, ISO formats with/without offset).
- **Follow the data.** Does what you POSTed come back identical in GET? In the list endpoint? In the UI that consumes it? Are calculated fields (totals, taxes, dates) correct?
- **Headers.** Wrong `Content-Type`, missing `Accept`, very large bodies, CORS behavior if the API is consumed by a browser.

Never run load, stress or flooding tests as part of an exploratory session — a handful of requests is enough to show a missing rate limit.

## Evidence for API defects

Each API defect should carry the exact request (method, URL, relevant headers with secrets masked, body) and the response (status, relevant headers, trimmed body). The `curl` command itself is the best reproduction step.
