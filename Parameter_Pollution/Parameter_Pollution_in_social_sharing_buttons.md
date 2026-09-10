# HTTP Parameter Pollution in Social Sharing Buttons

Some articles are shared via "share on Facebook/Twitter" buttons whose target URL is built by concatenating parameters from the current page's query string. Duplicated/polluted parameters let an attacker overwrite the shared link (`u`) and the shared text (`text`), so victim-shared posts carry attacker content. Usually low severity — but content injection into shares from the target's brand can be accepted as spam/phishing-adjacent.

Background on HPP in general: OWASP "Parameter Pollution" guidance and PayloadsAllTheThings *HTTP Parameter Pollution*.

## Hunting Checklist

1. Browse the target and find an article/blog page that has social-share links (Facebook, Twitter, addthis/sharethis widgets).
2. Inspect the share anchor's `href`: note the parameters it forwards — typically `u`/`url` and `text`/`title`.
3. Check whether the page *builds* the share URL from its own query string (dynamic share buttons are the vulnerable pattern).
4. Append a payload **after** a benign parameter using `?&...` and click each share button.
5. Observe the composed share dialog/post: if it contains the attacker URL or text, you have HPP content injection.
6. Also try duplicated originals (`?title=x&title=attacker`) to learn whether first or last value wins.

## Test Cases & Examples

Original walkthrough:

1. Browse through your target, say `https://target.com`.
2. Find an article or blog on the target website which has a link to share that blog on different social networks such as Facebook, Twitter, etc.
3. Say the article URL is `https://target.com/how-to-hunt`. Append the payload:

```text
?&u=https://attacker.com/vaya&text=another_site:https://attacker.com/vaya
```

so the final URL becomes:

```text
https://target.com/how-to-hunt?&u=https://attacker.com/vaya&text=another_site:https://attacker.com/vaya
```

4. Hit enter with the above URL and click "share" with a social media button.
5. Observe the content: if the share includes the payload (e.g. `https://attacker.com`), the target is vulnerable — otherwise try the next target.

Generic duplicate-parameter probe (first vs. last wins behavior):

```text
https://target.com/how-to-hunt?u=https://target.com/how-to-hunt&u=https://attacker.com
```

Share widget rendered from polluted parameters (example shape):

```html
<a href="https://twitter.com/intent/tweet?url=https%3A%2F%2Fattacker.com%2Fvaya&text=another_site%3Ahttps%3A%2F%2Fattacker.com%2Fvaya">Share</a>
```

## Bypass Techniques

- If the app sanitizes the first parameter, place the payload second (or vice-versa) — template engines and routers differ on first/last-wins behavior.
- URL-encode or double-encode the payload values (`%u` style, `%26`-embedded pairs) when the share builder decodes late.
- Swap parameter casing/near-names (`Url`, `URL`, `txt`) if the filter is a strict name match.
- When `u` is validated against the domain but `text` is not, inject at least the attacker string into `text` (still brand-abuse content injection).

## Tools & Resources

- Real-world report of this exact bug class: https://hackerone.com/reports/105953
- Recon one-liner for share endpoints in JS/HTML:

```bash
curl -s https://target.com/article | grep -Eio '(twitter\.com|intent|facebook\.com/sharer|addthis|sharethis)[^"]*'
```

- Author: **[KenAdams000](https://twitter.com/KenAdams000)**

## Report Tips

- Show the final share dialog/preview containing the attacker URL/text — the screenshot of the composed post *is* the impact proof.
- Frame it as brand/phishing content injection ("victims see attacker-controlled links shared under target.com's own buttons"), which is why some programs accept what others close as N/A.
- Keep it factual about severity: note "not severe by itself, but accepted as content/spam injection in prior reports" and link the public reference above.
