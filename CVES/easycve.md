# Easy CVEs via Research-Driven Hunting

New, low-hanging CVEs on forgotten public-facing tech (old Jira, WordPress, admin panels) get patched slowly — the "exploit" is often a single unauthenticated request. This workflow finds the target's exposed software, pins the *exact* version, then researches known CVEs until one reproduces. Typical impact: unauthenticated data disclosure or RCE-class findings that scanners flag but nobody verifies.

## Hunting Checklist

1. Grab all the subdomains.
2. Grab all alive domains.
3. Run Nuclei scans separately per template category, storing each result in its own file.
4. Read each output carefully, with patience.
5. Find interesting tech used by the target (e.g. Jira, WordPress).
6. Visit the page and check the exact version in use.
7. Google that version: `jira <version> exploit`.
8. Grep CVE IDs that look promising.
9. Search the CVE on Twitter: `CVE-XXXX-XXXX poc` / `CVE-XXXX-XXXX exploit`.
10. Google the CVE/exploit keywords for better PoCs or write-ups.
11. Test all candidates — if successful, report it!

## Test Cases & Examples

```bash
# 1. Grab all the subdomains
subfinder -d domain.com -o subs.txt

# 2. Grab all alive domains
httpx -l subs.txt -mc 200 -o alive.txt
# also useful here: httpx -l subs.txt -td -tech-detect output for step 5

# 3. Nuclei per category, one file each
nuclei -l alive.txt -t nuclei-templates/http/misconfiguration -o misconfigurations.txt
nuclei -l alive.txt -t nuclei-templates/http/exposed-panels -o exposed-panels.txt
nuclei -l alive.txt -t nuclei-templates/http/cves -o cves.txt
nuclei -l alive.txt -t nuclei-templates/http/technologies -o technologies.txt
```

Version confirmation for step 6 (once a panel is found):

```bash
# Version strings live in footers, page source, login pages, and JS/CSS asset paths
curl -sk https://jira.target.com/ | grep -oiE '(version|jira)[^<]{0,40}'
```

Research queries for steps 7–10 (fill the CVE ID from public sources — never from imagination):

```
<product> <version> exploit / cve
CVE-XXXX-XXXX poc
CVE-XXXX-XXXX exploit
CVE-XXXX-XXXX site:github.com poc
```

## Bypass Techniques

- **Scanner FPs dominate step 4:** a `technologies` hit ≠ vulnerable. Always pin the exact version string and compare it to the CVE's affected range before testing.
- **WAF in front:** confirm the panel really answers (fingerprint mismatch), or reach the origin (see [FindOrigin](../FindOriginIP/FindOrigin.md)) — but stay within program rules.
- **Patch-lag logic:** prioritize hosts whose version differs from the "main" instance — old copies on `test-`/`backup-`/`jira-internal` style subdomains are the usual wins.

## Tools & Resources

- [Nuclei + official template repo](https://github.com/projectdiscovery/nuclei-templates) — categories used above
- [subfinder](https://github.com/projectdiscovery/subfinder), [httpx](https://github.com/projectdiscovery/httpx) — PDDM discovery trio
- [CVE.ORG / NVD](https://cve.mitre.org/cgi-bin/cvekey.cgi) — authoritative CVE lookups (what's affected, which fixed version)
- [Exploit-DB](https://www.exploit-db.com) and GitHub PoC search for verified steps
- Twitter/X and blog writeups — fast PoC propagation (step 9)

## Report Tips

- Report only what you **reproduced safely** on the target: exact URL, version evidence, and the minimal PoC request/response — raw Nuclei output alone is usually closed as information for duplicates/FPs.
- Note the affected version *range* and that the same host is internet-exposed, so triage sees the business risk.
- Prefer read-only CVE verification (info disclosure) over RCE proof when an unauthenticated read proves the same issue.

### Authors

* [@Virdoex\_hunter](https://x.com/Virdoex_hunter)
* [@Psikoz\_Coder](https://x.com/Psikoz_Coder)
