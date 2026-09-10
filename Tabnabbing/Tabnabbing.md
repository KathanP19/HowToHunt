# Reverse Tabnabbing

Reverse tabnabbing lets a page opened via a `target="_blank"` link rewrite the originating tab's URL (`window.opener.location`), silently swapping the victim's still-logged-in page for a convincing phishing login. Many programs rate it low severity, but it is trivially easy to find and is a real credential-theft vector — worth hunting on every app that renders user-controlled links.

Related chain bug: [Open Redirect](../Open_Redirection/Open_Redirection_Hunting.md) (attacker-controlled link destinations).

## Hunting Checklist

1. Identify links the site renders with `target="_blank"` — especially user-controlled ones (forum posts, comments, profile fields, review text) where the attacker controls the `href`.
2. For each such link, check that the `rel` attribute does **not** contain `noopener` (or `noreferrer`) — without it, the opened page gets full `window.opener` access.
3. Bulk-search crawled HTML with:

```bash
curl -s https://target.com/blog-post | grep -o '<a [^>]*target="_blank"[^>]*>' | grep -v noopener
```

4. Or in Burp: engagement tool **Find links** / search engine over responses for `target="_blank"`, then audit the `rel` attribute.
5. Test as a logged-in victim: post a link (or use the existing external link) to your attacker page and see if the original tab can be navigated.
6. Check mitigations before reporting (see Bypass Techniques): modern browsers implied `rel="noopener"` for `target="_blank"` after the 2021 spec change — verify exploitation actually works in a current browser/agent used by the target's users.

## Test Cases & Examples

### Vulnerable link formats

```html
<a href="..." target="_blank" rel="" />
```

or simply:

```html
<a href="..." target="_blank" />
```

### Attacker-page payload

The page opened by the victim (`https://attacker.com/evil`) contains:

```javascript
window.opener.location = "http://evil.com";
```

### Exploit flow

1. Attacker posts a link to a website under his control containing the JS above.
2. He tricks the victim into visiting the link, which opens in a new browser tab.
3. The JS executes and the background tab is redirected to `evil.com` — most likely a phishing site.
4. If the victim reopens the background tab and doesn't look at the address bar, they may think they were logged out, because a login page appears.
5. The victim tries to log in again and the attacker receives the credentials.

## Bypass Techniques

- **Forced-opener contexts:** links injected via Markdown/BBCode renderers or rich-text fields that emit `rel="opener"` or strip `noopener` restore full exploitation even where default markup is safe.
- **`window.open` sinks:** pages that call `window.open(url)` without `noopener` in the feature string give the same control — search JS for `window.open(`.
- **iframe/`postMessage` variants:** an embedded frame can act on the parent (`parent.location`) when frames share no protection — pivot to [postMessage XSS](../XSS/post_message_xss.md) if the opener channel is richer.
- If the browser auto-mitigates (newer Chrome/Firefox), target contexts that opt back in: explicit `rel="opener"`, apps shipping Electron/WebView builds, or links opened from emails where the first party is your page anyway.

## Tools & Resources

- Burp Suite: Find links, search engine (`target="_blank"`), response grep
- Browser DevTools: console `window.opener.location = 'https://evil.com'` from the opened tab as a quick check
- Reference: OWASP [Reverse tabnabbing](https://owasp.org/www-community/attacks/Reverse_Tabnabbing) · HTML spec `rel=noopener` change (2021, browsers imply `noopener` for `target=_blank`)
- Author: **[@bolli95](https://github.com/bolli95)**

## Report Tips

- Even for low-severity programs: show the full chain with a working phishing page (original tab ends on your fake login) — proof of navigation, not just the missing `rel`.
- Prioritize user-controlled-link vectors (forum/comment/profile URL) over static site links; attacker-chosen `href` on authenticated pages is the strongest framing.
- Note affected user agents/browsers where it still fires, and mention the one-line fix (`rel="noopener noreferrer"`) to help triage accept it quickly.
