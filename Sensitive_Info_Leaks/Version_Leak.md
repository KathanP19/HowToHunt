# Version Leak (Server / Path Disclosure)

Version leaks expose the exact software (and its version) serving a site via error pages, `Server:` headers, and referenced asset paths — which lets an attacker map to known CVEs instantly. Small to fix, but it's a real information-disclosure finding and a force multiplier for further testing.

## Hunting Checklist

1. Fingerprint the obvious places first: HTTP response headers and HTML/JS/CSS references (source maps, asset paths often embed absolute server paths).
2. Walk the directories you can see referenced and note which return 403 vs 404 vs 200.
3. Trigger error pages with malformed path segments (`%00`, `%0a`, `%2e`, random junk after a valid path).
4. Compare the error/403 pages for server banners (name + version).
5. Check common verbose endpoints if the stack suggests them (`/server-status`, `/server-info`, phpinfo).
6. Map any discovered version to known CVEs (see [Easy CVEs](../CVES/easycve.md)) and report.

## Test Cases & Examples

Original manual method:

1. Go to the target, e.g. `https://redacted.com`.
2. Open **View Page Source**.
3. Check for paths and directories referenced in the HTML (CSS/JS/img URLs).
4. Follow one, e.g. `https://redacted.com/theme/css/file.css`.
5. Try to visit each parent directory and note whether it's accessible (a **403** means the directory exists).
6. Add special bytes such as `%0`, `%m`, `%2e` — e.g. `https://redacted.com/%0theme` — and check the response: error pages often show the running server name and version information.
7. Also inspect CSS file URLs — they sometimes contain server paths.

Commands:

```bash
# Header/banner fingerprinting
curl -sI https://example.com

# Malformed segment tricks that can trigger version-disclosing error pages
curl -sk 'https://example.com/%00' 
curl -sk 'https://example.com/%0a'
curl -sk 'https://example.com/%2e'
curl -sk 'https://example.com/random/%3f.jsp'

# Verbose status pages (only if enabled by misconfiguration)
curl -sk https://example.com/server-status
curl -sk https://example.com/server-info

# Watch asset paths for embedded absolute paths
curl -s https://example.com | grep -oE '(src|href)="[^"]*"' | grep -v '^.*="http'
```

## Bypass Techniques

- **403s from a WAF vs the origin:** repeat requests through an origin IP (see [Finding Origin IP](../FindOriginIP/FindOrigin.md)) — the origin's error page leaks the real version.
- If `%00` is filtered, alternate encodings/malformed bytes (`%0a`, `%2e`, overlong UTF-8) can still reach the server's error handler — test each, filtering is usually incomplete.
- Custom error pages may hide the banner, but path disclosure can still appear in **stack traces** triggered by odd parameters (`?`, `]`, quotes).

## Tools & Resources

- `curl -I` / browser DevTools Network tab — headers
- [WhatWeb](https://github.com/urbanadventurer/WhatWeb) — version fingerprinting from markup/headers
- [httpx](https://github.com/projectdiscovery/httpx) `-td` tech detect; banner grabbing with `nmap -sV` / `masscan`
- Related: [CVES/easycve.md](../CVES/easycve.md), [FindOriginIP](../FindOriginIP/FindOrigin.md)

## Report Tips

- Version leaks alone are usually informative/low unless you **tie the exact version to publicly known CVEs or an exposed status page** — include that chain in the report.
- Provide the request that triggered the disclosure and the full leaking response; recommend suppressing `Server` tokens and custom error pages.
