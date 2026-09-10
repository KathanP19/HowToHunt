# Default Credentials & Exposed Admin Panels

Applications, devices, and third-party services frequently ship with well-known default accounts (admin/admin, tomcat/tomcat, etc.) or expose an admin console at a guessable path or subdomain. Left unchanged, these give an attacker direct administrative control. Impact is typically High–Critical: full application takeover, data access, and configuration changes — and even an unauthenticated *login page* exposure is often reportable when combined with defaults or weak lockout.

## Hunting Checklist

1. Fuzz for admin consoles on the main site and each subdomain — common paths:

```text
https://www.target.com/admin
https://www.target.com/admin-console
https://www.target.com/console
https://admin.target.com
https://admin-console.target.com
https://console.target.com
```

2. Extend with product-specific panels: `/wp-admin`, `/administrator` (Joomla), `/system/login`, `/manager/html` (Tomcat), `/manager/status`, `/phpmyadmin/`, `/dashboard`, `/login`, `/cpanel`, `/web-console`, and staging/dev variants (`stage.admin.target.com`, `beta.target.com/admin`).
3. Fingerprint what powers the login page (Wappalyzer/BuiltWith, page source, cookie names). If the admin console is served by a **third-party service**, search the vendor's documentation for factory default credentials.
4. Third-party service login URLs often follow the format `https://target.<TPS-Name>.com/login` (e.g. Okta, WP and similar SaaS panels) — hunt these subdomains explicitly.
5. Test vendor-documented default credentials *carefully* (1–3 attempts only, respect lockout), using lists from SecLists.
6. If the login page returns 403/404 to you but exists (e.g. linked, or found via recon), test the header-injection bypass below.
7. Check for IP-restriction bypasses: `X-Forwarded-For` / `X-Real-IP` spoofing on admin paths.
8. Look for default credentials disclosed in exposed files (backups, `.env`, JS bundles) via the Sensitive Info Leaks workflow.

## Test Cases & Examples

```http
GET /admin HTTP/1.1
Host: www.target.com
```

If the panel is powered by a third-party product, confirm with:

```bash
whatweb https://target.okta.com/login
curl -sk https://www.target.com/manager/html | grep -i "Apache Tomcat"
```

Guessing defaults (from vendor docs / SecLists `Passwords/Default-Credentials/`):

```http
POST /admin/login HTTP/1.1
Host: www.target.com
Content-Type: application/x-www-form-urlencoded

username=admin&password=admin
```

## Bypass Techniques

**Getting to a forbidden login page via header injection** — when you are forbidden (403) from reaching the admin login page, inject rewriting headers under the `Host` header and compare responses in Burp:

```http
GET /anything HTTP/1.1
Host: www.target.com
X-Original-URL: /admin
X-Rewrite-URL: /admin
```

Some stacks honor only one of the headers; try each alone and combined, and vary the path casing (`/Admin`, `/ADMIN`). Related tricks where the panel is IP-allowlisted:

```http
GET /admin HTTP/1.1
Host: www.target.com
X-Forwarded-For: 127.0.0.1
X-Real-IP: 127.0.0.1
```

## Tools & Resources

- **Tools:** ffuf/dirb + SecLists (`Discovery/Web-Content/common.txt`, `Passwords/Default-Credentials/`), Burp Repeater, Wappalyzer, whatweb, nuclei (`miscellaneous`, `default-login` template tags).
- [HackerOne report #192074](https://hackerone.com/reports/192074)
- [HackerOne report #174883](https://hackerone.com/reports/174883)
- [HackerOne report #398797](https://hackerone.com/reports/398797)
- [OWASP: Testing for Default Credentials (OTG-AUTHN-002)](https://www.owasp.org/index.php/Testing_for_default_credentials_(OTG-AUTHN-002))
- [OWASP: Testing for Default or Guessable User Account (OWASP-AT-003)](https://www.owasp.org/index.php/Testing_for_Default_or_Guessable_User_Account_(OWASP-AT-003))

## Report Tips

- Don't stop at "admin page found" — demonstrate the login (default creds accepted, or lockout absent) and what an admin can do; exposed defaults = Critical, exposed-but-protected panel = informational/low unless chained.
- Never modify or delete data once logged in; capture one benign read (version, user list) as proof.
- Note remediation: remove/change defaults, restrict panel by IP + MFA, and delete unused third-party subdomain panels.

## Author

[@e11i0t_4lders0n](https://twitter.com/e11i0t_4lders0n)
