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

### Reports (Hackerone)

#### Resolved

- [IDOR to delete images from other stores](https://hackerone.com/reports/404797)
- [IDOR in changing shared file name](https://hackerone.com/reports/547663)
- [User uploaded portfolio files can be accessed by any user even after deleted](https://hackerone.com/reports/300179)
- [IDOR and statistics leakage in Orders](https://hackerone.com/reports/544329)
- [I.D.O.R To Order,Book,Buy,reserve On YELP FOR FREE (UNAUTHORIZED USE OF OTHER USER'S CREDIT CARD)](https://hackerone.com/reports/391092)
- [IDOR allow access to payments data of any user](https://hackerone.com/reports/751577)
- [IDOR allow to extract all registered email](https://hackerone.com/reports/302485)
- [IDOR at https://account.mackeeper.com/at/load-reports/profile/<profile_id> leaks information about devices/licenses](https://hackerone.com/reports/783117)
- [IDOR bug to See hidden slowvote of any user even when you dont have access right](https://hackerone.com/reports/661978)
- [IDOR on update user preferences](https://hackerone.com/reports/854290)
- [idor on upload profile functionality](https://hackerone.com/reports/741683)
- [IDOR to view User Order Information](https://hackerone.com/reports/287789)
- [IDOR with Geolocation data not stripped from images](https://hackerone.com/reports/906907)
- [Replace other user files in Inbox messages](https://hackerone.com/reports/322661)

### Author

* [@harsha0x01](https://twitter.com/harsha0x01)
* [@klaus](https://twitter.com/klaus_dev)
