# Finding the Origin IP Behind a WAF

Web Application Firewalls (Cloudflare, AWS WAF, Akamai, etc.) shield the true server by fronting all traffic, but the **origin IP** is frequently discoverable from DNS history, certificates, subdomains, and email headers. Knowing the origin lets a tester bypass every WAF rule entirely and reach admin interfaces hidden behind the proxy. For a program, an exposed origin is a critical finding: the whole protection layer becomes trivially skippable.

## Hunting Checklist

1. **Confirm a WAF is present** and identify the provider (`dig`, org of the IP, cookies/headers).
2. **Pull historical DNS records** (SecurityTrails, DNSDumpster) for A records used before WAF adoption.
3. **Enumerate subdomains** — dev/staging/test/beta hosts often sit outside the WAF and resolve to (or near) the origin.
4. **Harvest certificate data** — search Censys/Shodan by CN/SAN to find every host a cert was deployed on.
5. **Test every candidate IP** by requesting the target hostname directly against it.
6. **Read email headers** from triggered transactional emails for internal `Received`/`X-Originating-IP` chains.
7. **Run dedicated tooling** (CloudFail/CloudFlair) to automate the above.
8. **Verify the origin is real** before reporting (content match + server fingerprint).

## Test Cases & Examples

### Identifying the presence of a WAF

```bash
# Get the IP address
dig +short example.com

# Check the organization
curl -s https://ipinfo.io/IP | jq -r '.org'
```

**Common WAF indicators:**
- AWS WAF/CloudFront: `AWSALB` and `AWSLBCORS` cookies
- Cloudflare: IP ownership resolves to Cloudflare, Inc.
- Other WAFs: specific signatures or response headers

### 1. Historical DNS records

Historical DNS records often reveal IPs used before WAF implementation:

- **SecurityTrails DNS history** — `https://securitytrails.com/domain/example.com/dns`, export historical A records, then:

```bash
grep -E -o "([0-9]{1,3}[\.]){3}[0-9]{1,3}" dns_history.txt | sort -u > potential_ips.txt
```

- **DNSDumpster** — https://dnsdumpster.com generates network maps; look for non-WAF IPs in the results.

### 2. Subdomain enumeration

Development or staging environments often lack proper WAF protection:

```bash
# Find subdomains and their IPs
subfinder -silent -d example.com | dnsprobe -silent | awk '{ print $2 }' | sort -u > subdomain_ips.txt
```

Focus on subdomains like: `dev.example.com`, `staging.example.com`, `test.example.com`, `beta.example.com`.

### 3. SSL certificate information

Certificates reveal all domains/IPs where they're deployed:

- **Censys method:**
  1. Search for certificates using your target domain
  2. Select "Certificates" in the input field and search for your domain
  3. Review each certificate and click "Explore" → "IPv4 Hosts"
  4. Collect all associated IPs

- **Shodan method:**

```bash
# Search by Common Name (CN)
shodan search 'ssl.cert.subject.CN:"example.com"'

# Search in all certificate fields (broader)
shodan search 'ssl:"example.com"'
```

> **Note:** Verify results manually — they may include CDN/proxy IPs. SAN (Subject Alternative Name) fields are generally more reliable than CN.

### 4. Direct IP testing

For each potential IP, test whether it serves the target hostname:

```bash
# Test single IP
curl -s -k -H "Host: example.com" https://POTENTIAL_IP/

# Test multiple IPs
for ip in $(cat potential_ips.txt); do
  org=$(curl -s https://ipinfo.io/$ip | jq -r '.org')
  title=$(timeout 2 curl -s -k -H "Host: example.com" https://$ip/ | pup 'title text{}')
  echo "IP: $ip | Title: $title | Org: $org"
done
```

### 5. Email headers analysis

Transactional emails from the target often contain internal IP information:

1. Trigger emails from the target (register, password reset, newsletters)
2. Examine headers, particularly: `Return-Path`, `Received`, `X-Originating-IP`

### 6. Specialized tools

```bash
# CloudFail
git clone https://github.com/m0rtem/CloudFail.git
cd CloudFail
pip install -r requirements.txt
python3 cloudfail.py -t example.com

# CloudFlair
git clone https://github.com/christophetd/CloudFlair
cd CloudFlair
pip install -r requirements.txt
python3 cloudflair.py example.com
```

### Verifying the origin IP

1. Compare response content with the WAF-protected site
2. Look for server fingerprints (headers, error pages)
3. Check for administrative interfaces or panels not accessible via the WAF

## Bypass Techniques

- **Once the origin is found, all WAF bypass becomes unnecessary** — send payloads directly to the IP (mind SNI: use `--resolve`/`Host:` rather than plain `https://IP`).
- Combine techniques: an IP that only appears in *old* DNS history may still answer today, and vice versa.
- Rate-limit/timing evasion: space out scans against the origin — its own host-based IDS may be the only defense.
- **Don'ts that get bans:** never DDoS or brute-force the origin; verifying reachability with a single request is enough for a report.

## Tools & Resources

- [SecurityTrails](https://securitytrails.com), [DNSDumpster](https://dnsdumpster.com) — historical/passive DNS
- [Censys](https://censys.io), [Shodan](https://www.shodan.io) — certificate & banner search
- [CloudFail](https://github.com/m0rtem/CloudFail), [CloudFlair](https://github.com/christophetd/CloudFlair) — automation
- `pup` (HTML parsing), `ipinfo.io` (org lookup), `subfinder`/`dnsprobe` (PDDM tools)
- References: [Navisec: A Pentester's Guide — Unmasking WAFs and Finding the Source](https://delta.navisec.io/a-pentesters-guide-part-5-unmasking-wafs-and-finding-the-source/), [Detectify: Bypassing Cloudflare WAF with the Origin Server IP Address](https://blog.detectify.com/2019-07-31/bypassing-cloudflare-waf-with-the-origin-server-ip-address/)

## Report Tips

- Best proof for triage: one `curl` against the origin IP returning identical content to the WAF-fronted hostname — that demonstrates the full bypass in seconds.
- Frame impact as "WAF bypass + direct exposure of all unpatched origin vulnerabilities and any hidden admin panels", not just "IP found".
- Document all discovered IPs and verification status; recommend the fix (firewall the origin to WAF IPs only, rotate exposed certs/keys).

## Credits

### Original Author
* [maverickNerd](https://x.com/maverickNerd)

### Contributors
* [nagarajcruze](https://github.com/nagarajcruze)
* [www](https://github.com/www)

---
*Enhanced and reformatted for HowToHunt repository by [remonsec](https://x.com/remonsec)*
