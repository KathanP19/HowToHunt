# Web Source Code Review Tips

Reviewing a web application's source (client-side code you can read, or code you're given/leaked) exposes vulnerability classes that black-box testing misses: weak crypto, dangerous functions, forgotten debug endpoints, and hardcoded secrets. It's also the fastest way to *discover* attack surface — every endpoint, parameter, and validation gap becomes visible. Typical impact: authentication bypass, info leaks, and injection sinks you can then prove with live requests.

## Hunting Checklist

1. **Important functions first** — read authentication, password reset, state-changing actions, and sensitive-info reads; then how they interact with other functionality; audit the rest last.
2. **Follow user input** — trace HTTP params, headers, paths, DB entries, file reads/uploads to the sinks they reach.
3. **Grep for hardcoded secrets** — API keys, encryption keys, DB passwords.
4. **Hunt dangerous functions & outdated dependencies** — language-specific sinks + dependency version review.
5. **Read developer comments & debug features** — TODOs marking bugs, hidden debug routes, privilege-escalation paths.
6. **Map configuration files & exposed `.git`** — infra details; reconstruct full source from leaked VCS data.
7. **List hidden paths, deprecated endpoints, in-development endpoints** — compare route definitions against what the UI exposes.
8. **Audit crypto/hashing choices** — weak algorithms rarely show up in black-box tests.
9. **Check missing security controls on input** — validation gaps, weak/regex-bypassable checks, missing cookie flags.
10. **Study conditionals & complex/verbose functions** — obscure bugs live where the logic is gnarly.

## Test Cases & Examples

### 1. Important functions first
When reading source code, focus on functions such as authentication, password reset, state-changing actions, and sensitive info reads (what is most important depends on the application). Then review how these components interact with other functionality. Finally, audit other less sensitive parts of the application.

### 2. Follow user input
User input — HTTP request parameters, HTTP headers, request paths, database entries, file reads, file uploads — provides the entry points attackers exploit. Tracing it often surfaces critical bugs directly (XXE, XSS, SQL injection).

### 3. Hardcoded secrets and credentials

```bash
# Keyword sweep (adapt to the language): keys, secrets, passwords, tokens
grep -rniE '(api[_-]?key|secret|password|passwd|token|private[_-]?key)' src/

# Entropy-style sweep: long hex/base64 blobs depending on key format in use
grep -rnoE '([A-Fa-f0-9]{32,}|[A-Za-z0-9+/]{40,}={0,2})' src/
```

### 4. Dangerous functions & outdated dependencies
Grep for the dangerous functions of the language in use (e.g. `eval`, `exec`, `system`, `os.system`, `subprocess`, `child_process.exec`, `pickle.loads`, `unserialize`, raw SQL string concatenation) and review dependency version lists against known-vulnerable releases (see [Easy CVEs workflow](../CVES/easycve.md)).

### 5. Comments, debug features, config files, `.git`
Developer comments point out obvious mistakes; hidden debug functionality → privilege escalation; config files → infrastructure intel; an exposed `.git` directory → full source reconstruction:

```bash
# Classic live-target checks for accidentally published internals
curl -s https://target.com/.git/HEAD
curl -s https://target.com/web-inf/web.xml   # example for Java stacks
```

### 6. Hidden / deprecated / in-development endpoints
Users never see these in the UI, but if they work they can yield auth bypass or sensitive data leaks — enumerate route definitions and test each.

### 7. Weak cryptography or hashing

```bash
grep -rniE '(ecb|des|rc4|md4|md5|sha1\b)' src/   # weak ciphers/hashes as the primary protection
```

### 8. Missing security checks on user input
Read the application's own documentation and test all edge cases you can think of — a great resource for edge cases to consider is **PayloadsAllTheThings** (github.com/swisskyrepo/PayloadsAllTheThings).

### 9. Missing cookie flags
Look for cookies set without `HttpOnly` and `Secure` (and missing `SameSite`) in framework config and response code.

### 10. Unexpected behavior, conditionals, complex functions
Pay special attention to gnarly conditionals and unnecessarily complex/verbose functions — obscure bugs are consistently found there.

## Bypass Techniques

- When client-side validation is the only control, it's bypassed with a proxy (repeat requests directly with Burp) — but the *real* finding is whether the server-side code shares the same weak checks; verify sink-by-sink.
- Obfuscated JS bundles: pretty-print in DevTools, restore source maps if shipped (`//# sourceMappingURL=...`), then grep the reconstructed source like above.

## Tools & Resources

- ripgrep / IDE global search — the greps above
- [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings) — edge-case inspiration for #8
- [GitHub dork lists](../Sensitive_Info_Leaks/Github-dorks.md) for applying checks #3–#5 at search scale; TruffleHog / [git-hound](https://github.com/tillson/git-hound) for automated secret review
- Framework route/dependency audits: `npm audit`, `pip-audit`, `bundle audit`, OWASP dependency-check (standard tooling)

## Report Tips

- Findings discovered "by reading" still need a **live proof of impact** — show the actual request/response that abuses the weak code path.
- Frame severity by what the flaw enables (auth bypass > info leak > style issue); pure dependency-version claims need a working CVE match for *this* deployment.

### Authors

* [@harsha0x01](https://twitter.com/harsha0x01)
