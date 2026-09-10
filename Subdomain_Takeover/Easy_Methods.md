# Subdomain Takeover — Easy Methods (Quick Wins)

A fast, tool-driven path to subdomain takeover findings: enumerate → dig CNAMEs → let a scanner claim-check them. These two community methods are deliberately minimal — full background, fingerprints, and proof-of-takeover etiquette live in `Subdomain_Takeover.md`; the advanced wildcard/TLD variant is in `Sub_or_top_level_domain_takeover.md`. Typical impact/severity when a method lands: High–Critical (attacker content on trusted origin).

## Hunting Checklist

1. Grab all subdomains of the target.
2. Resolve them and pull the CNAME entries.
3. Keep only records pointing at a third-party provider (e.g. `sub.example.com CNAME x.aws.com`).
4. Check whether the resource/subdomain is actually down (unclaimed) — provider error page = candidate.
5. Register/claim the target on the host provider; if it accepts your claim, you have taken over the subdomain. PoC it cleanly and report.

## Test Cases & Examples

### Method 1 — by @Virdoex_hunter

```bash
# 1) grab all subdomains, e.g.
subfinder -d flaws.cloud | tee -a domains.txt

# 2-3) one-liner: resolve every host and capture the output
cat domains.txt | while read domain; do dig $domain; done | tee -a digs.txt

# 4) grab all the CNAME entries
cat digs.txt | grep CNAME

# 5) find a domain pointed to a third-party domain, e.g.
#    sub.example.com CNAME x.aws.com
# 6) check whether the main subdomain is down
# 7) go to the host provider the CNAME points to and register the resource —
#    if you registered it, congrats: you have taken over the subdomain.
```

### Method 2 — by @WhoIs1nVok3r

```bash
# Step-1: collect all subdomains (assetfinder, subfinder, chaos — needs API key)
assetfinder --subsidiary target.com > unresolved
subfinder -d target.com >> unresolved

# Step-2: sort out duplicate URLs
cat unresolved | sort -u | tee -a resolved

# Step-3: pass to a takeover scanner (subzy, subjack, ...)
subzy -targets resolved

# Step-4 (alternative): probe the live hosts, then run templates
cat resolved | httpx | tee -a hosts
cat hosts | nuclei -t nuclei-templates/vulnerabilities -o nuclei.txt -v
```

## Bypass Techniques

Not the focus here — the quick wins mostly *miss* when: wildcard CNAMEs mask dangling names, providers require multi-step ownership verification, or the CNAME target is a purchasable external domain. Those cases go to the main guide (Bypass section) and the TLD doc.

## Tools & Resources

From Method 2 (all Project Discovery unless noted):

- https://github.com/projectdiscovery/nuclei
- https://github.com/projectdiscovery/subfinder
- https://github.com/projectdiscovery/httpx
- https://github.com/projectdiscovery/nuclei-templates
- https://github.com/projectdiscovery/chaos-client
- https://github.com/haccer/subjack
- https://github.com/LukaSikic/subzy

Plus `dig` (bind-utils) and can-i-take-over-xyz from the main guide.

## Report Tips

- Even for a "quick win", the report needs: the enum command/output snippet, the CNAME line, the provider's unclaimed error page, and your claimed PoC page + release confirmation.
- Never auto-claim at scale (mass `subzy` then mass registration): claim one, report, stop — bulk registration trips legal review on most programs.

## Author

[@Virdoex_hunter](https://twitter.com/Virdoex_hunter) · [@WhoIs1nVok3r](https://twitter.com/WhoIs1nVok3r)
