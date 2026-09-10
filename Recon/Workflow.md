# Recon Workflow

Recon is the foundation of every bug-bounty hunt: mapping the target's external attack surface before testing anything. A thorough, repeatable workflow uncovers forgotten assets (staging apps, acquired brands, stray APIs) where bugs hide because nobody else looked. Typical impact: initial access to unscoped-but-owned systems, exposed admin panels, and bugs missed by shallow hunters.

## Hunting Checklist

1. **IP space discovery** — find the target's ASNs and owned IP ranges (never test IPs that only belong to a VPS/CDN provider).
2. **TLDs, acquisitions & relations** — map sibling domains, acquired companies, and alternate TLDs that share the same infrastructure.
3. **Subdomain enumeration** — run the passive → active → permutation pipeline (see [subdomain_enumeration.md](subdomain_enumeration.md)).
4. **Fingerprinting** — identify technologies, servers, and versions on every live asset.
5. **Dorking** — mine GitHub/Google/Shodan for leaked secrets and exposed assets (see [Github_Dorking.md](Github_Dorking.md) and the [Sensitive_Info_Leaks](../Sensitive_Info_Leaks/Github-dorks.md) docs).
6. **Content discovery** — brute-force/fuzz paths, files, and backup artifacts.
7. **Parameter discovery** — find hidden request parameters that gate undocumented functionality.

## Test Cases & Examples

### ASN / IP-space discovery

```bash
# ASN discovery via bgp.he.net — search the org name, note all ASNs listed
# https://bgp.he.net

# ASN for a domain's IP via Team Cymru whois
whois -h whois.cymru.com $(dig +short example.com)

# Enumerate IPs belonging to an ASN with nmap
nmap --script targets-asn --script-args targets-asn.asn=<ASN Number>
```

> **NOTE:** Be careful — sometimes the ASN returned belongs to a VPS provider (e.g. DigitalOcean) rather than the target. Don't work on those IPs unless explicitly in scope.

### Company / asset intelligence with Amass

```bash
# Gather company intel (organisation name, not domain)
amass intel -org <Organisation name(not domain)>

# Subdomains derived from the target's ASNs
amass intel -asn <ASN_number>

# Reverse-whois: find other domains registered by the same entity
amass intel -d example.com -whois
```

Additional ASN lookups: [ARIN whois](https://whois.arin.net), [IPinfo](https://ipinfo.io).

### Discovering brands, acquisitions & related orgs

Sources to mine for sister domains and acquired companies:

- Wikipedia
- Crunchbase — https://www.crunchbase.com
- Owler — http://owler.com
- AcquiredBy — https://acquiredby.co
- LinkedIn
- BuiltWith — https://builtwith.com (tech + related-site discovery)
- Google dork for shared copyright footers:

```
intext:"copyright © org_name"
```

- Shodan dork by HTTP favicon hash (same favicon ⇒ same infrastructure, even on unknown domains):

```
http.favicon.hash:<hash>
```

The favicon hash can be computed with [FavFreak](https://github.com/devanshbatham/FavFreak).

## Tools & Resources

- [bgp.he.net](https://bgp.he.net) — ASN/IP-range lookup
- [whois.arin.net](https://whois.arin.net), [ipinfo.io](https://ipinfo.io) — IP ownership intel
- [Amass](https://github.com/owasp-amass/amass) — `intel` module for org/ASN/reverse-whois
- [FavFreak](https://github.com/devanshbatham/FavFreak) — favicon hash generation for Shodan/Censys pivoting
- [BuiltWith](https://builtwith.com/), Crunchbase, Owler, AcquiredBy — corporate footprint mapping
- Related docs: [Subdomain Enumeration](subdomain_enumeration.md), [GitHub Dorking](Github_Dorking.md), [Finding Origin IPs](../FindOriginIP/FindOrigin.md)

## Report Tips

- Recon itself is rarely a bug, but "forgotten" assets found via this workflow (staging sites, acquired brands with weak protection) are often where critical findings live.
- When reporting bugs on non-obvious assets, include the pivot chain (ASN → IP → reverse DNS → cert/favicon match → subdomain) so triage can verify ownership quickly.

### Author

[Mr._fr3qu3n533](https://twitter.com/mr_fr3qu3n533)
