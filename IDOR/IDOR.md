# IDOR (Insecure Direct Object Reference)

An IDOR is an access control flaw where the application relies on user-supplied input to reference an object directly — a picture, a comment, PII attached to a user, or an entire department's data. Because authorization is checked (if at all) on the reference itself, an attacker can modify the value and bypass authorization to read or mutate other users' resources such as database records or files. Typical impact ranges from PII disclosure and statistics leakage to full account takeover and financial abuse (e.g. spending another user's saved card), so findings are usually rated medium to critical depending on the object and verb involved. IDORs exist everywhere IDs are used, so always test when you see one — even GUIDs or "encrypted" ids; look for leaks of valid ids (public profiles?) and patterns so you can generate your own and run them through Burp Intruder.

## Hunting Checklist

1. Create two accounts (attacker + victim) if possible; otherwise enumerate users first.
2. Identify every request that carries an id/hash/token-like parameter; check whether the endpoint is private or public.
3. Figure out what the app uses for authorization (JWT, API keys, cookies, tokens) by swapping a high-privilege credential for a low-privilege one and observing what the server responds with.
4. Understand how IDs, hashes and the API are used — read the API documentation if one is exposed.
5. Every time you see a new API endpoint that receives an object ID from the client, ask:
   - Does the ID belong to a private resource? (e.g. `/api/user/123/news` vs `/api/user/123/transaction`)
   - What IDs belong to me?
   - What roles exist in the API? (e.g. `user`, `driver`, `supervisor`, `manager`)
6. Replace the parameter value with the other account's id and check whether anything reads or changes their data.
7. If the id looks unguessable, look for leaks of it (public profiles, error messages, other endpoints) and test whether predictable patterns let you generate ids for Intruder.
8. Automate cross-account verification (e.g. Autorize) instead of blindly incrementing numbers until you hit PII.

## Test Cases & Examples

**Type 1 — parameter value retrieves a database record**

```http
GET /somepage?invoice=12345 HTTP/1.1
```

**Type 2 — parameter value performs an operation**

```http
GET /changepassword?user=someuser HTTP/1.1
```

**Type 3 — parameter value retrieves a file system resource**

```http
GET /showImage?img=img00011 HTTP/1.1
```

**Type 4 — parameter value gates application functionality**

```http
GET /accessPage?menuitem=12 HTTP/1.1
```

### TC1: Add IDs to requests that don't have them

```http
GET /api/MyPictureList → GET /api/MyPictureList?user_id=<other_user_id>
```

Pro tip: find parameter names to try by deleting or editing other objects and watching which parameter names the app uses.

### TC2: Replace parameter names

```http
Instead of: GET /api/albums?album_id=<album_id>
Try:        GET /api/albums?account_id=<account_id>
```

Tip: the Burp extension Paramalyzer helps here by remembering every parameter you have passed to a host.

### TC3: Supply multiple values for the same parameter (HPP)

```http
GET /api/account?id=<your_account_id>&id=<admin_account_id> HTTP/1.1
```

This is HTTP parameter pollution — the framework may bind the first value to the auth check and the second to the query, landing you in the admin's account.

### TC4: Try changing the HTTP request method

```http
POST /api/account?id=<your_account_id>   →   PUT /api/account?id=<your_account_id>
```

Tip: try switching POST and PUT and see if you can upload something to another user's profile. For RESTful services, try changing GET to POST/PUT/DELETE to discover create/update/delete actions.

### TC5: Change the request's content type

Access controls may be inconsistently implemented across content types:

```http
POST /api/chat/join/123 HTTP/1.1
Content-Type: application/xml

<user>test</user>
```

```http
POST /api/chat/join/123 HTTP/1.1
Content-Type: application/json

{"user": "test"}
```

Don't forget alternative and less common values like `text/xml`, `text/x-json`, and similar.

### TC6: Change the requested file type (if Ruby)

```http
GET /user_data/2341 HTTP/1.1       → 401 Unauthorized
GET /user_data/2341.json HTTP/1.1  → 200 OK
```

Tip: experiment by appending different file extensions (`.json`, `.xml`, `.config`) to requests that reference a document.

### TC7: Use numeric IDs where the app expects non-numeric ones

There may be multiple ways of referencing the same object and access controls are only applied to one:

```http
username=user1           → username=1234
account_id=7541A92F-0101-4D1E-BBB0-EB5032FE1686 → account_id=5678
album_id=MyPictures      → album_id=12
```

### TC8: Wrap the ID in an array

```http
POST /api/object/read HTTP/1.1
Content-Type: application/json

{"id":19}      → 401 Unauthorized
{"id":[19]}    → 200 OK
```

### TC9: Wildcard ID

These are exciting (and simple) to find in the wild — try replacing an ID with a wildcard:

```http
GET /api/users/<user_id>/ → GET /api/users/*
```

### TC10: Pay attention to new features

If you find a newly added feature (e.g. uploading a profile picture for an upcoming charity event) that calls:

```http
POST /api/CharityEventFeb2021/user/pp/<ID> HTTP/1.1
```

the app may not enforce access control on this new feature as strictly as on its core features.

### Other checks

- For high-impact IDOR, find hidden parameters on sensitive endpoints using Arjun and Param Miner: `/settings/profile`, `/user/profile`, `/user/settings`, `/account/settings`, `/username`, `/profile`, and any payment endpoint.
- Don't forget create/update/delete on objects that are publicly readable but shouldn't be writable — can you `PUT /api/products` and change a price?
- If the endpoint is named like `/api/users/myinfo`, check `/api/admins/myinfo`.
- If it is a numeric id, test through a large range of numbers instead of just guessing nearby values.
- Difficulty is easy — don't blindly test for changing numbers until you get PII, tools can do that part for you.

## Bypass Techniques

When a straight id swap returns 401/403:

- Add parameters onto the endpoint:

```http
GET /api_v1/messages                          → 401
GET /api_v1/messages?user_id=victim_uuid      → 200
```

- HTTP parameter pollution:

```http
GET /api_v1/messages?user_id=ATTACKER_ID                 → 401 Unauthorized
GET /api_v1/messages?user_id=ATTACKER_ID&user_id=VICTIM_ID → 200 OK

GET /api_v1/messages?user_id=YOUR_USER_ID[]&user_id=ANOTHER_USERS_ID[]
```

- Add `.json` to the endpoint, if the app is built in Ruby:

```http
GET /user_data/2341        → 401 Unauthorized
GET /user_data/2341.json   → 200 OK
```

- Test outdated API versions:

```http
GET /v3/users_data/1234 → 403 Forbidden
GET /v1/users_data/1234 → 200 OK
```

- Wrap the ID with an array:

```http
POST /api/object/read HTTP/1.1
Content-Type: application/json

{"id":111}      → 401 Unauthorized
{"id":[111]}    → 200 OK
```

- Wrap the ID in a nested JSON object:

```http
POST /api/object/read HTTP/1.1
Content-Type: application/json

{"id":111}              → 401 Unauthorized
{"id":{"id":111}}       → 200 OK
```

- JSON parameter pollution (duplicate key in body):

```http
POST /api/get_profile HTTP/1.1
Content-Type: application/json

{"user_id":<legit_id>,"user_id":<victim's_id>}
```

- Send a wildcard (`*`) instead of an ID — rare, but sometimes works.
- Replace the request method (GET/POST/PUT).

### Escalating / chaining IDORs

1. A low-impact IDOR (e.g. changing someone else's display name) chained with a chat/message renderer = stored XSS.
2. If the endpoint requires a UUID, chain it with info-disclosure endpoints that leak UUIDs to bypass the unguessable-id problem.
3. Read/leak primitives chain into ATO: enumeration of emails, payment data, then credential/OTP flows.
4. If none of these work, get creative and ask around.

## Tools & Resources

**Automation / Burp:**
- [PimpMyBurp #1 - PwnFox + Autorize: The perfect combo to find IDOR](https://blog.yeswehack.com/yeswerhackers/pimpmyburp-pwnfox-autorize-find-idor/)
- [Automating BURP to find IDORs](https://medium.com/cyberverse/automating-burp-to-find-idors-2b3dbe9fa0b8)
- [Finding Broken Access Controls](https://threat.tevora.com/finding-broken-access-controls/)
- Burp extensions: Autorize, Paramalyzer; CLIs: Arjun, Param Miner (hidden parameters).

**References:**
- [Everything You Need to Know About IDOR (Insecure Direct Object References)](https://medium.com/@aysebilgegunduz/everything-you-need-to-know-about-idor-insecure-direct-object-references-375f83e03a87)
- [Finding more IDORs - Tips and Tricks | Aon](https://www.aon.com/cyber-solutions/aon_cyber_labs/finding-more-idors-tips-and-tricks/)
- [Learn about Insecure Object Reference (IDOR) | BugBountyHunter.com](https://www.bugbountyhunter.com/vulnerability/?type=idor)
- [WSTG - Testing for Insecure Direct Object References](https://owasp.org/www-project-web-security-testing-guide/v42/4-Web_Application_Security_Testing/05-Authorization_Testing/04-Testing_for_Insecure_Direct_Object_References.html)
- [IDOR (Insecure Direct Object Reference) notes](https://notes.mufaddal.info/web/idor)
- [What I learnt from reading 220* IDOR bug reports](https://medium.com/@Sm9l/what-i-learnt-from-reading-220-idor-bug-reports-6efbea44db7)
- [Twitter tip: finding high-impact IDOR via hidden parameters](https://twitter.com/M0_SADAT/status/1361289751597359105)
- [Attack vector ideas thread](https://twitter.com/swaysThinking/status/1301663848223715328)

**Writeups:**
- [Full account takeover worth $1000 — think out of the box](https://mokhansec.medium.com/full-account-takeover-worth-1000-think-out-of-the-box-808f0bdd8ac7)
- [All About Getting First Bounty with IDOR](https://infosecwriteups.com/all-about-getting-first-bounty-with-idor-849db2828c8)
- [Hunting IDORs for fun and profit (Part 1)](https://codeburst.io/hunting-insecure-direct-object-reference-vulnerabilities-for-fun-and-profit-part-1-f338c6a52782)
- [IDOR that allowed me to takeover any user's account](https://vedanttekale20.medium.com/idor-that-allowed-me-to-takeover-any-users-account-129e55871d8)
- [All About IDOR Attacks](https://betterprogramming.pub/all-about-idor-attacks-64c4203b518e)
- [Access developer tasks list of any Facebook Application (GraphQL IDOR)](https://amineaboud.medium.com/access-developer-tasks-list-of-any-of-facebook-application-graphql-idor-62307c5e5b34)

**Example resolved HackerOne reports:**
- [IDOR to delete images from other stores](https://hackerone.com/reports/404797)
- [IDOR in changing shared file name](https://hackerone.com/reports/547663)
- [User uploaded portfolio files accessible by any user even after deletion](https://hackerone.com/reports/300179)
- [IDOR and statistics leakage in Orders](https://hackerone.com/reports/544329)
- [I.D.O.R To Order, Book, Buy, reserve on YELP for FREE (unauthorized use of other user's credit card)](https://hackerone.com/reports/391092)
- [IDOR allow access to payments data of any user](https://hackerone.com/reports/751577)
- [IDOR allow to extract all registered email](https://hackerone.com/reports/302485)
- [IDOR at load-reports/profile/<profile_id> leaks devices/licenses](https://hackerone.com/reports/783117)
- [IDOR bug to see hidden slowvote of any user](https://hackerone.com/reports/661978)
- [IDOR on update user preferences](https://hackerone.com/reports/854290)
- [IDOR on upload profile functionality](https://hackerone.com/reports/741683)
- [IDOR to view User Order Information](https://hackerone.com/reports/287789)
- [IDOR with Geolocation data not stripped from images](https://hackerone.com/reports/906907)
- [Replace other user files in Inbox messages](https://hackerone.com/reports/322661)

## Report Tips

- Show the authorization boundary being crossed with two accounts: "Account A can read/modify Account B's <object> — proof: B's PII/order/payment data in the response." Enumerated-volume demos (N ids → N records) raise severity.
- Frame impact by the object and verb: read = disclosure (PII, payment data), write = integrity/ATO (email change, preferences, spending another user's saved card), delete = data destruction/DoS on victim content.
- If the id was a UUID/GUID, explain how you obtained or generated valid ids — triagers reject "theoretical" IDOR without a realistic id source.
- Include the chained impact (e.g. IDOR → stored XSS, IDOR → ATO) when the direct impact looks low.

## Authors

- [KathanP19](https://twitter.com/KathanP19)
- [@harsha0x01](https://twitter.com/harsha0x01)
- [@klaus](https://twitter.com/klaus_dev)
