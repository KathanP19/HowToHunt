## Testing for IDOR/Broken object level authorization:

Difficulty: Easy 

Tips: Don't blindly test for changing numbers till you get PII, tools can do this for you

**Finding IDOR Attack Vectors Ideas:**

1. What  do they use for authorization?(JWT, API Keys, cookies, tokens) Tip: Find this out by replacing high privaledge authorization with lower privaledge authorization and seeing what the server responds with
2. Understand how they use ID's, hashes, and their API. Do this by looking at the API Documentations if they have one.

***Every time you see a new API endpoint that receives an object ID from the client, ask yourself the following questions:***

- Does the ID belong to a private resource? (e.g /api/user/123/news vs  /api/user/123/transaction)
- What are the IDs that belong to me?
- What are the different possible roles in the API?(For example — user, driver, supervisor, manager)

## Bypassing Object Level Authorization:

- Add parameters onto the endpoints for example, if there was

```html
GET /api_v1/messages --> 401
vs 
GET /api_v1/messages?user_id=victim_uuid --> 200
```

- HTTP Parameter pollution

```html
GET /api_v1/messages?user_id=VICTIM_ID --> 401 Unauthorized
GET /api_v1/messages?user_id=ATTACKER_ID&user_id=VICTIM_ID --> 200 OK

GET /api_v1/messages?user_id=YOUR_USER_ID[]&user_id=ANOTHER_USERS_ID[]
```

- Add .json to the endpoint, if it is built in Ruby!

```html
/user_data/2341 --> 401 Unauthorized
/user_data/2341.json --> 200 OK
```

- Test on outdated API Versions

```html
/v3/users_data/1234 --> 403 Forbidden
/v1/users_data/1234 --> 200 OK
```

* Wrap the ID with an array.

```html
{“id”:111} --> 401 Unauthriozied
{“id”:[111]} --> 200 OK
```

* Wrap the ID with a JSON object:

```html
{“id”:111} --> 401 Unauthriozied

{“id”:{“id”:111}} --> 200 OK
```

* JSON Parameter Pollution:

```html
POST /api/get_profile
Content-Type: application/json
{“user_id”:<legit_id>,”user_id”:<victim’s_id>}
```

- Try to send a wildcard(*) instead of an ID. It’s rare, but sometimes it works.
- If it is a number id, be sure to test through a large amount of numbers, instead of just guessing
- If endpoint has a name like /api/users/myinfo, check for /api/admins/myinfo
- Replace request method with GET/POST/PUT
- Use burp extension autorize
- If none of these work, get creative and ask around!

## Escalating/Chaining with IDOR's Ideas:

1.  Lets say you find a low impact IDOR, like changing someone elses name, chain that with XSS and you have stored XSS!
2. If you find IDOR on and endpoint, but it requires UUID, chain with info disclosure endpoints that leak UUID, and bypass this!
3. If none of these work, get creative and ask around!

### Reference
[https://twitter.com/swaysThinking/status/1301663848223715328](https://twitter.com/swaysThinking/status/1301663848223715328)

### Author
* [@harsha0x01](https://twitter.com/harsha0x01)
