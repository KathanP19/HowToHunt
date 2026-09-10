# Moodle

Moodle is the dominant open-source LMS, and thousands of university/school instances are public — which makes it ideal for *mass hunting*: the same feature surface (forums, assignments, profiles, quizzes) repeats across every deployment, so one XSS pattern becomes many reports. Typical findings are stored/reflected XSS and access-control issues between courses; impact on an LMS is high because grades, personal data, and teacher/admin sessions live there.

## Hunting Checklist

1. **Find instances:** search public subdomains/hosts for LMS patterns (`moodle.`, `learn.`, `campus.`, `lms.`, `staging.` on university domains); detect with Wappalyzer/whatweb (`Moodle`) or the `/login/index.php` + `/my/` URL scheme and footer "Moodle" link.
2. **Fingerprint the version** where exposed (`/admin/version.php`, about/plugin listings) and compare against the official security announcements list for that branch — verify advisories, never guess.
3. **Map the auth surface:** `/login/signup.php` (self-registration on/off), guest course access, "forgot password" behavior, and default `admin`/`admin` on fresh or staging installs (test at most once; never brute force).
4. **Enumerate objects:** `/course/view.php?id=N`, `/user/view.php?id=N`, `/user/profile/view.php?id=N`, `/mod/forum/view.php?id=N` — walk IDs as an anonymous and free-registered user to find information disclosure between courses.
5. **Hunt stored XSS** in every rich-text field that renders for other users: profile fields/description, forum posts & replies, assignment/feedback comments, chat/wiki/glossary entries (this is the mass-hunting staple — see reference under Tools).
6. **Hunt reflected XSS** in search boxes and `id`/`q` params of `/mod/*/...` pages, and in error/validation messages.
7. **Check plugin paths** (`/mod/<plugin>/`, `/blocks/<plugin>/`) — third-party plugins are where the weak spots usually are.
8. **Note exposed artifacts:** leftover backup files, debug output, install/upgrade pages left reachable on staging copies.

## Test Cases & Examples

```http
GET /login/signup.php HTTP/1.1
Host: lms.target.com

GET /course/view.php?id=1 HTTP/1.1
Host: lms.target.com

GET /user/view.php?id=2 HTTP/1.1
Host: lms.target.com
```

Stored-XSS sweep in profile/forum/assignment fields (use HTML/editor contexts, fire the payload as another user):

```html
<img src=x onerror=alert(document.domain)>
<svg/onload=alert(document.domain)>
<a href="javascript:alert(1)">x</a>
```

Mass-hunting flow: register on N in-scope instances, submit the same payload in each renderer (Atto/HTML editor vs plain text), and record which ones execute.

## Bypass Techniques

- Rich-text editors strip tags on *save* — bypass by submitting during edit (URL-encoded payload), switching editors (Atto ↔ TinyMCE ↔ Markdown/HTML format), or injecting attributes rather than new tags (`" onmouseover=...`).
- Text filters convert `&colon;`/entities — try `<img src=x onerror&colon;alert(1)>` and `javascript:` with entities (`jav&#x61;script:`).
- WAF/CDN in front of the LMS: test the raw application host from recon and staging copies first.
- CSP blocks inline handlers on some instances: pivot to `javascript:` hrefs, or DOM sinks like `#` fragments in the editor.

## Tools & Resources

- **Mass Hunting XSS — Moodle** (the writeup this guide is built around): https://dewangpanchal98.medium.com/mass-hunting-xss-moodle-ed4b50c82516
- whatweb / Wappalyzer / nuclei for detection; Burp for renderer/replay of stored payloads; `subfinder`/`assetfinder` for instance discovery.
- Official: Moodle security announcements page (verify advisories per version branch).

## Report Tips

- Stored XSS in an LMS is High: show it executing in a *victim context* (teacher/admin viewing the student post/profile) and the session/grade data at risk.
- For mass-hunting patterns, file one report per program/instance with its own PoC screenshot — don't bundle unrelated tenants in one submission.
- Cross-course access issues: frame as "student of Course A reads Course B enrolments/grades" with two accounts as proof; never touch real student data beyond reading your own planted content.

## Author

[@th3.d1p4k](https://twitter.com/DipakPanchal05)
