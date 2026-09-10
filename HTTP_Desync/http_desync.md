# HTTP Desync / Request Smuggling

HTTP request smuggling interferes with the way a site processes the sequence of HTTP requests received from one or more users — "Request smuggling vulnerabilities are often critical in nature, allowing an attacker to bypass security controls, gain unauthorized access to sensitive data, and directly compromise other application users." — PortSwigger. It arises when a front-end proxy and a back-end server disagree on where one request ends and the next begins (typically CL vs `Transfer-Encoding` handling). Typical impact: cache poisoning, hijacking other users' requests/sessions, ACL bypass, web-cache deception — frequently critical.

## Hunting Checklist

1. Identify a proxy in front of the app (CDN/WAF/load balancer + origin): response header signatures, `X-Cache`, timeouts, differing behavior between front-end and back-end errors. Desync needs two components that parse HTTP differently.
2. Test any endpoint — smuggling can exist on non-endpoints too, but the impact is much higher on sensitive endpoints (login, account, payment, admin paths).
3. In Burp Repeater, try timing-based detection payloads (CL.TE, TE.CL) against both GET and POST; a consistent multi-second hang on the *second* identical request (or timeout) indicates a desync.
4. Compare responses with the smuggled content visible in the next request's response (content-based detection) when timing is noisy.
5. Try `Transfer-Encoding` obfuscation variants — both peers must "accept" the header while disagreeing on its meaning.
6. Also check HTTP/2-only paths (H2.TE / H2.CL desync) when the target speaks H2: forward smuggling via H2 downgrades to H1 at the proxy.
7. Confirm with a tool for coverage: smuggler across the subdomain set, Burp Request Smuggler in Repeater.
8. Once confirmed, look for a chain: front-end cache poisoning, stealing session tokens via a queued response, forcing XSS via smuggled `User-Agent`/header handling, path-based ACL bypass.

## Test Cases & Examples

### CL.TE detection (front-end uses Content-Length, back-end uses Transfer-Encoding)

Send the request twice (or send a second innocuous request after it); a hang/timeout on the follow-up indicates the back-end is still waiting inside the smuggled chunk stream.

```http
POST / HTTP/1.1
Host: example.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 6
Transfer-Encoding: chunked

0

G
```

Front-end forwards 6 bytes (`0\r\n\r\nG`); the back-end (TE) sees the final chunk `0` and treats the trailing `G` as the start of a new request, waiting for its headers — your next request gets absorbed (or times out).

### TE.CL detection (front-end uses Transfer-Encoding, back-end uses Content-Length)

```http
POST / HTTP/1.1
Host: example.com
Content-Type: application/x-www-form-urlencoded
Transfer-Encoding: chunked
Content-Length: 4

8
G
0
```

The front-end (TE) reads chunk `8` and consumes `G`, `0`, and more as chunk data; the back-end (CL) consumes only 4 bytes as the body and treats the remainder as a new request — subsequent responses are desynchronized (visible as your later request's output arriving early, or a hang).

### TE.TE detection (both peers TE, but one can be confused)

Obfuscate `Transfer-Encoding` so one peer ignores it while the other honors it. Variants (from the "HTTP Desync Attacks" paper):

```
Transfer-Encoding: xchunked
Transfer-Encoding : chunked
Transfer-Encoding: chunked
Transfer-Encoding : chunked      (trailing space)
Transfer-Encoding:
Transfer-Encoding: x
Transfer-Encoding:chunked
Transfer-Encoding:[[tab]]chunked
Transfer-Encoding: chunked
Transfer-Encoding: identity
```

### CL.0 / TE.0

When the back-end treats *any* invalid CL/TE combination as "body length 0", smuggling works even against servers normally immune to CL.TE/TE.CL — these are the "new era" desyncs from the PortSwigger research post (see Resources): a header-less body is simply never consumed by the back-end, so the bytes remain in its input buffer and the next user's request inherits them.

### Chaining ideas

- Queue a crafted response in the back-end buffer so it is served to the next victim (session-token theft → ATO).
- Smuggle a header line into the next request to poison cached objects.
- Cross-site scripting via smuggled payloads in header handling (e.g., User-Agent).

## Bypass Techniques

- WAF/proxy header checks: use TE.TE obfuscation variants above, duplicate headers, or header-case/tab tricks.
- CL.0/TE.0 to defeat "we use the same server on both sides" defenses.
- H2CL desync: smuggle over HTTP/2 (where CL/TE are forbidden) down to an H1 back end.
- Padding large bodies to mask timing noise in low-traffic targets.

## Tools & Resources

Tools:

- Smuggler (defparam): https://github.com/defparam/smuggler

```bash
cat alive_urls.txt | python3 smuggler.py -m GET/POST     # either GET or POST
# or
python3 smuggler.py -u https://example.com -m GET/POST
```

- Burp Request Smuggler extension (BApp store): https://github.com/PortSwigger/http-request-smuggler

Learning:

- https://paper.seebug.org/1049/ (recommended)
- PortSwigger research — HTTP Desync Attacks: Request Smuggling Reborn: https://portswigger.net/research/http-desync-attacks-request-smuggling-reborn
- PortSwigger finding guide: https://portswigger.net/web-security/request-smuggling/finding
- PortSwigger exploiting guide: https://portswigger.net/web-security/request-smuggling/exploiting
- PortSwigger lab track: https://portswigger.net/web-security/request-smuggling
- Video: https://youtu.be/gzM4wWA7RFo

HackerOne reports:

- https://hackerone.com/reports/737140
- https://hackerone.com/reports/867952
- https://hackerone.com/reports/498052
- https://hackerone.com/reports/526880
- https://hackerone.com/reports/771666
- https://hackerone.com/reports/753939
- https://hackerone.com/reports/648434
- https://hackerone.com/reports/740037

Writeups:

- https://medium.com/@ricardoiramar/the-powerful-http-request-smuggling-af208fafa142
- https://medium.com/cyberverse/http-request-smuggling-in-plain-english-7080e48df8b4
- https://medium.com/@cc1h2e1/write-up-of-two-http-requests-smuggling-ff211656fe7d
- https://medium.com/bugbountywriteup/crossing-the-borders-the-illegal-trade-of-http-requests-57da188520ca

## Report Tips

- State the misparses proven (CL.TE / TE.CL / TE.TE / CL.0), the proxy stack inferred, and give reproducible timing evidence (identical request #1 = fast, #2 = hang, #3 = both absorbed).
- Chain to concrete impact: stolen session of a subsequent request, poisoned cache entry served to other users, or internal ACL bypass — unchained timing-only proofs still warrant high severity given the class.
- Never demonstrate on other real users; use your own second account/proxy session and clearly mark synthetic follow-up requests.

---
*Author: [Neutron__](https://twitter.com/Neutron__)*
