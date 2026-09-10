# Subdomain Enumeration

Subdomain enumeration is the core of attack-surface mapping — essential for wildcard-enabled scope programs, and for finding forgotten dev/staging/CI assets that the team itself has lost track of. If you resolve subdomains nobody else sees, you get first look at the bugs living on them (takeovers, exposed panels, debug endpoints).

## Hunting Checklist

1. **Passive collection** — query as many data sources as possible (certificate transparency, search engines, DNS aggregators, pastes) using API-key-driven tools; costs no traffic to the target.
2. **Merge & de-duplicate** — combine outputs into one sorted list; keep sources separate so you can judge reliability later.
3. **Wildcard detection** — before trusting results, check the domain for wildcard DNS, or every random name will "resolve".
4. **Active brute-force** — resolve wordlist candidates against the target to find records not published anywhere; tune the wordlist to the org's naming conventions (dev, stage, api, internal, brand names).
5. **Permutation** — mutate already-known names (`api` → `api-dev`, `staging-api`, `test2`) and re-resolve; these frequently hit undocumented environments.
6. **Resolve & verify** — convert surviving names to IPs, then port-scan to find live services on odd ports.
7. **Fingerprint the survivors** — hand off to httpx/nmap and continue the [recon workflow](Workflow.md).

## Test Cases & Examples

### Passive enumeration

Community-standard tools that consume API keys and aggregate dozens of sources:

- **Subfinder** — `subfinder -d example.com -o subs.txt`
- **Amass** — `amass enum -passive -d example.com`
- **Assetfinder** — `assetfinder --subs-only example.com`
- **Findomain** — cert-transparency focused, very fast

### Wildcard check

```bash
# Resolve two random, unlikely names; identical/mirrored answers = wildcard DNS
dig +short a1b2c3d4.example.com
dig +short e5f6a7b8.example.com
```

### Active brute-force

In this stage you perform brute-forcing against the target to see if a word from your wordlist resolves as a valid subdomain:

- **ShuffleDNS** — `shuffledns -d example.com -list subs.txt -resolver resolvers.txt -wordlist subdomains.txt`
- **Aiodnsbrute** — async brute-forcer

### Permutation

Play around with known subdomains: mutate/combine words and check whether they still resolve as valid names:

- **AltDNS**
- **DNSGen + ShuffleDNS** (generate permutations, then brute-resolve them)

### Port scanning

Convert domains into IP addresses:

```bash
while read l; do ip=$(dig +short $l|grep -oE "\b([0-9]{1,3}\.){3}[0-9]{1,3}\b"|head -1);echo "[+] '$l' => $ip";echo $ip >> ips.txt;done < domains.txt
```

Use masscan for faster results:

```bash
masscan -p1-65535 -iL ips.txt --max-rate 1800 -oG output.log
```

Or use [Naabu](https://github.com/projectdiscovery/naabu) or [RustScan](https://github.com/RustScan/RustScan/).

## Bypass Techniques

- **Rate limiting / source throttling:** rotate resolvers (public DNS lists for ShuffleDNS) and run passive sources over several days instead of hammering one API.
- **Private/internal names:** they won't appear in CT logs — brute-force and permutation are the only ways in.
- There's a lot more you can do; the above are the community-standard approaches.

## Tools & Resources

- [SEF](https://github.com/remonsec/SEF) — automated framework that chains this whole workflow
- [Pentester.Land subdomain enumeration cheatsheet](https://pentester.land/cheatsheets/2018-11-14/subdomains-enumeration-cheatsheet.html)
- [Subdomain enumeration (2019) — 0xpatrik](https://0xpatrik.com/subdomain-enumeration-2019/)
- [Subdomain enumeration — smarter — 0xpatrik](https://0xpatrik.com/subdomain-enumeration-smarter/)
- [Bug bounty recon — faster port scan](https://rootsploit.com/bug-bounty-recon-faster-port-scan/)

## Report Tips

- A subdomain that resolves but is dead/unclaimed may be a **subdomain takeover** — prove it with a working PoC (CNAME to an available resource) rather than reporting the name alone.
- For undocumented environments (staging, internal panels), framing matters: explain business risk (weak credentials, test data, production parity) — see [Subdomain_Takeover](../Subdomain_Takeover/Subdomain_Takeover.md).

### Author

[Mehedi Hasan Remon](https://twitter.com/remonsec)
[Rishi Choudhary](https://twitter.com/0xRyuk)
