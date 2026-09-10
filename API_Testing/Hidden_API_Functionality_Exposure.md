# Hidden API Functionality Exposure

APIs are the backbone of modern applications, carrying data between internal systems and external consumers — but teams frequently expose functionality that was never meant to be public: interactive API docs (Swagger/OpenAPI), debug/admin endpoints, unreleased version routes, and entire hidden endpoint trees. Discovering these gives attackers a ready-made map of the attack surface and, when the docs or endpoints lack authentication, direct access to sensitive functionality and data. Impact ranges from full API schema disclosure to unauthenticated CRUD on backend objects.

## Hunting Checklist

1. Capture normal application traffic and note the API base paths and versioning style (`/api/v1/...`).
2. Check for exposed documentation UIs (Swagger UI, OpenAPI JSON/YAML, ReDoc, Postman/GraphQL equivalents) at common paths.
3. If no docs are found, brute-force endpoint paths with API-specific wordlists (steps below).
4. Compare status codes/lengths to filter noise; chase every HTTP 200 on a hidden endpoint.
5. Recursively fuzz interesting endpoints (`/v1/users` → `/v1/users/export`, `/v1/users/admin`, ...).
6. Sweep API versions: an endpoint blocked on `/api/v3/` may work unchanged on `/api/v1/` or `/api/v2/`.
7. Test the same endpoints unauthenticated vs authenticated to find missing auth checks on hidden functionality.
8. Mine JS bundles for API paths/parameters the UI never renders (routes for admin/staff features).
9. Correlate with other modules: found endpoints should also be tested for IDOR, BOLA, mass assignment, rate limiting.

## Test Cases & Examples

### 1. Swagger / API documentation exposure

Try the common documentation paths on the API host:

```http
GET /swagger/index.html HTTP/1.1
GET /swagger-ui.html HTTP/1.1
GET /swagger-ui/index.html HTTP/1.1
GET /api/swagger-ui.html HTTP/1.1
GET /api-docs HTTP/1.1
GET /openapi.json HTTP/1.1
GET /v2/api-docs HTTP/1.1
GET /v3/api-docs HTTP/1.1
GET /documentation HTTP/1.1
GET /apispec.json HTTP/1.1
```

A 200 on any of these hands you every endpoint, parameter, and model of the API — including hidden admin/debug functions. From the docs (or `swagger.json`/`openapi.json`), extract the endpoint list and feed it into your fuzzer.

### 2. Dictionary attack / endpoint brute force

Steps to perform this attack:

1. Capture a request into Burp, send it to Repeater and Intruder.
2. Add the endpoint path as the payload position and load an API wordlist.
3. First use a dictionary attack with [SecLists](https://github.com/danielmiessler/SecLists) against the endpoint.
4. Then use a customized list or one of the API-specific lists (see Tools below).
5. Start the attack and filter results for HTTP 200.
6. Once there is an HTTP 200 OK, start a recursive scan on that same endpoint for juicy information like swagger docs and so on.
7. Alternative method: change the API version and bruteforce the same endpoint.

Same flow with a CLI fuzzer:

```bash
ffuf -u 'https://target.com/api/v1/FUZZ' \
  -w SecLists/Discovery/Web-Content/api/api-endpoints.txt \
  -mc 200,201,204,401,403,405 -fs 0
```

### 3. API version sweeps

```http
GET https://redacted.com/api/v1/{Endpoint}   → 403/404
GET https://redacted.com/api/v2/{Endpoint}   → 200 OK
```

```bash
for v in v1 v2 v3 beta internal admin; do
  curl -sk -o /dev/null -w "%{http_code} /api/$v/users\n" "https://target.com/api/$v/users"
done
```

### 4. Response-code triage

Hidden endpoints often differ from 404 noise: a `401/403` on an unlinked endpoint proves it exists, `405` means it exists with another verb (try GET/POST/PUT/DELETE/OPTIONS), and `200` on a GET to a normally-POSTed route can leak data or docs.

## Bypass Techniques

- **Rate limits / anti-bot on fuzzing:** there is a minimum request limit per unauthenticated session (no API key) — so utilize a manual approach as much as you can for high-value paths, then automate the rest of the scanning.
- **Version-based WAF gaps:** `v1` routes are commonly excluded from the rules that protect the current `v3` API.
- **Trailing slash / dot / case variants:** `/api/users` vs `/api/users/` vs `/API/Users` can evade naive path matching.
- **Docs JSON without the UI:** `/swagger-ui.html` may be blocked while `/swagger.json`, `/openapi.json`, or `/v2/api-docs` is still served.

## Tools & Resources

- [Assetnote API wordlists](https://wordlists.assetnote.io/) — high-quality, continuously refined API endpoint/prefix lists.
- [Net-hunter121/API-Wordlist](https://github.com/Net-hunter121/API-Wordlist)
- [SecLists](https://github.com/danielmiessler/SecLists) — `Discovery/Web-Content/api/`
- [OWASP API Security Top 10](https://owasp.org/www-project-api-security/) — project-BOLA and API inventory context.
- Burp Suite (Intruder/Repeater), ffuf, or Kiterunner (routes built from API specs).
- [Swagger Editor](https://editor.swagger.io/) — render any discovered `spec.json`/`spec.yml`.

## Report Tips

- Show the exact exposed URLs and, for docs, a redacted screenshot proving they describe the *production* API (endpoint count, auth flows listed).
- Escalate the finding: don't just report Swagger UI — pick one undocumented endpoint it revealed and demonstrate real impact (unauthorized data read, admin action).
- Quantify enumeration: "successfully enumerated N hidden endpoints / M admin functions, all unauthenticated" gives triage a measurable scope statement.
- Note if the exposed API schema leaks internal implementation details (stack trace shapes, internal ids, PII field names) that aid further attacks.

## Contributor

- [N3T_hunt3r](https://twitter.com/N3T_hunt3r)
