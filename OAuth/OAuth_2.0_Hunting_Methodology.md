# OAuth 2.0 Hunting Methodology

OAuth 2.0 is the de-facto authorization framework behind "Login with Google/Facebook/..." SSO and API delegation. Because it stitches together a client app, an authorization server and a browser redirect chain, small implementation mistakes — missing `state`, weak `redirect_uri` validation, unvalidated tokens on the callback — are common and directly lead to account takeover or forced account linking.

In OAuth there are two main flows/grant types to recognize: **Authorization code flow** and **Implicit flow**.

> Note: if the OAuth service uses the authorization code flow there is little to no chance of finding a bug *in the front-channel*; if it uses the implicit flow there is a good chance of finding bugs.

## Hunting Checklist

1. Trigger the "Login with …" flow and capture it; the authorization endpoint may be named differently (`/auth`, `/authorization`…) — identify it by its parameters: `client_id`, `redirect_uri`, `response_type`, `scope`, `state`.
2. **Determine the grant type** from `response_type` (`code` vs `token`) and what arrives at the callback (authorization code vs access token).
3. **Check the `state` parameter** on the linking/callback request — its absence means CSRF (forced profile linking).
4. **Test `redirect_uri` validation** — swap in an external URL you control (webhook.site), then alternate paths/subdomains on the same host.
5. If implicit flow: find the **POST that trades the access token + user info for a session cookie** in HTTP history and test whether the server validates the token against the provider.
6. Replay/manipulate: capture the code/token callback, **drop the original request** so the code stays unused, then reuse it.
7. Deliver the crafted URL (or `<iframe>` it on your site) and complete the flow.
8. Also review: token leakage via Referer/fragments, scope creep, and whether the callback validates the `state` it originally set.

## Test Cases & Examples

### Identifying the grant type

**Authorization code flow — authorization request** (client asks the OAuth service's `/authorization` endpoint for access to specific user data):

```http
GET /authorization?client_id=12345&redirect_uri=https://client-app.com/callback&response_type=code&scope=openid%20profile&state=ae13d489bd00e3c24 HTTP/1.1
Host: oauth-authorization-server.com
```

`response_type=code` — that code is later exchanged (backend-side, invisible to your proxy) for the access token. After the user logs in and consents, they are redirected to the `redirect_uri` with the code as a query parameter:

```http
GET /callback?code=a1b2c3d4e5f6g7h8&state=ae13d489bd00e3c24 HTTP/1.1
Host: client-app.com
```

Factors that determine authorization code flow:
- initial authorization request has `response_type=code`
- the `/callback` request contains the authorization code as a parameter

**Implicit flow — authorization request** starts the same way, only `response_type=token`:

```http
GET /authorization?client_id=12345&redirect_uri=https://client-app.com/callback&response_type=token&scope=openid%20profile&state=ae13d489bd00e3c24 HTTP/1.1
Host: oauth-authorization-server.com
```

On consent, the redirect carries the access token and token-specific data as a **URL fragment** instead of a code:

```http
GET /callback#access_token=z0y9x8w7v6u5&token_type=Bearer&expires_in=5000&scope=openid%20profile&state=ae13d489bd00e3c24 HTTP/1.1
Host: client-app.com
```

Factors that determine implicit flow:
- initial authorization request has `response_type=token`
- the `/callback` request contains the access token as a parameter

### Method 1 — Auth bypass in OAuth implicit flow

- To log the user in every time with OAuth, the client app sends a **POST containing user info (email, username) + access token** to generate a session cookie.
- Find that POST request in HTTP history.
- Usually in implicit flow the server **doesn't validate the access token** — so change the email/username parameters to impersonate another user and bypass authentication.

### Method 2 — Forced profile linking (missing `state` → CSRF)

Similar to a traditional CSRF attack (impact may be lower, but still valid).

1. Sign in with a social media profile, log in there, and get redirected back; next login is instant — capture this with Burp.
2. In HTTP history find the request similar to `GET /auth?client_id[...]`; its `redirect_uri` sends the authorization code to something like `/oauth-linking`.
3. **Check if the `state` parameter is present.** If not, the server has no way to verify the info came from the same user → vulnerable; absence of `state` here is itself a vulnerability.
4. Exploit: copy the URL of the request in Burp and **drop the request** so the code isn't used. Turn off intercept and log out.
5. Send the link to the victim, or host it as an iframe on your site:

```html
<iframe src="https://target.com/auth?client_id=[...]&redirect_uri=https://target.com/oauth-linking?code=[...]"></iframe>
```

6. When the victim's browser loads the iframe, the OAuth flow completes and **your social media profile gets attached to their account** (then you can log into the victim's account via "your" linked provider).

### Method 3 — Account hijacking via `redirect_uri`

1. Complete the OAuth sign-in, log out, log back in (instant) and capture it.
2. Find the most recent authorization request `GET /auth?client_id=[...]` in HTTP history.
3. Check `redirect_uri` and try to change it — if you can redirect to an external site, good; if not, try different endpoints on the same website.
4. If there is an open redirect, change `redirect_uri` to your **webhook.site** link and follow the redirect.
5. Check webhook.site logs for a request containing the **authorization code**.
6. Send the request URL to the victim (or iframe it as in Method 2) with `redirect_uri` set to your webhook site — when they click, their authorization code lands in your logs.
7. Replay the stolen code in the callback request; the rest of the OAuth flow completes automatically and you are logged in as the victim (e.g. the admin user).

## Bypass Techniques

- `redirect_uri` allowlist filtering (standard checks to try):
  - append the payload path/subdomain behind the allowlisted host: `https://target.com.evil.com`, `https://target.com@evil.com`
  - allowlisted-suffix tricks: `https://target.com.evil.com` vs `https://evil.com/?x=target.com`, and `https://evil.com/target.com`
  - different endpoint on same host, `//evil.com`, `/\evil.com`, URL-encoded (`%2f`, double encoding), added port, `https:` vs `http:` mismatch
- Case/parameter confusion: duplicate `redirect_uri`, send it in POST body as well as query.
- `state` present but not validated: fixate a `state` from your own session and reuse it in the victim's flow (login CSRF).
- Implicit-flow fragment leakage: access token may leak via Referer, history or an XSS'd postMessage handler on the callback page.

## Tools & Resources

- Burp Suite (HTTP history, Repeater — drop the request to keep the code unused)
- [webhook.site](https://webhook.site) / interactsh — catch leaked codes/tokens
- [PortSwigger Web Security Academy — OAuth](https://portswigger.net/web-security/oauth)
- [OAuth 2.0 Threat Model and Security Considerations (RFC 6819)](https://datatracker.ietf.org/doc/html/rfc6819)

**Mindmap** (see [README](README.md)):
![OAuth mindmap](https://pbs.twimg.com/media/EZ1WqmcXYAAqwSH?format=jpg&name=900x900)

## Report Tips

- Method 2 (forced linking): demonstrate you can attach your identity to a victim and then log in as them → account takeover; a missing `state` alone is the root cause to highlight.
- Method 3 (code leak): show a full chain — crafted URL → victim click → code at webhook.site → session established; that removes any "theoretical" doubt for triage.
- Implicit-flow token validation bypass: prove login as another user with an untouched/expired/fake access token plus the session cookie received.

**Author:** [Pyr0sec](https://twitter.com/Pyr0sec)
