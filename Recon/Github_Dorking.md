# GitHub Dorking

Developers leak API keys, tokens, credentials, internal configs, and whole dumps into GitHub constantly — often inside the target org's own repositories or under employee accounts. Systematic GitHub dorking turns public code search into an initial-access and information-disclosure goldmine. Typical impact: valid third-party API keys, private registry access, database dumps, and sometimes direct access to production systems.

> This doc covers the *workflow*. The dork corpora live in the Sensitive_Info_Leaks section — don't duplicate them here:
> - **[Github-dorks.md](../Sensitive_Info_Leaks/Github-dorks.md)** — curated essential dorks (files, languages, keys, usernames, extensions)
> - **[Github_dorks_all.md](../Sensitive_Info_Leaks/Github_dorks_all.md)** — exhaustive reference (full secret/keyword/regex corpus + long-tail dorks)
> - **[Github_Recon_Method.md](../Sensitive_Info_Leaks/Github_Recon_Method.md)** — step-by-step method with real HackerOne report examples

## Hunting Checklist

1. **Scope the org on GitHub** — find the official org (`org:name`), then enumerate its public repos and contributors.
2. **Dork by org name, not just org: qualifier** — employees leak target names in personal repos, snippets, and configs; search the company/brand/product name as a quoted string combined with secret keywords.
3. **Enumerate humans** — pull employee usernames from LinkedIn/Twitter/commits and search their repos, gists, and profile fields (see [Github_Recon_Method.md](../Sensitive_Info_Leaks/Github_Recon_Method.md) step 2).
4. **Run the curated dorks** — work through [Github-dorks.md](../Sensitive_Info_Leaks/Github-dorks.md) with `"org_name"` prepended.
5. **Deep-scan with the full corpus** — for automated sweeps, feed [Github_dorks_all.md](../Sensitive_Info_Leaks/Github_dorks_all.md) into a script/tool rather than searching manually.
6. **Validate before touching** — confirm any found key/token actually works *and* belongs to the program; check recent commits first (leaks are often already rotated).
7. **Automate persistence** — schedule re-runs; new leaks appear daily.

## Test Cases & Examples

### Org-scoped search templates

Replace `org_name` with the target company/product name and run each (full syntax reference: [Github-dorks.md](../Sensitive_Info_Leaks/Github-dorks.md)):

```
"org_name" password
"org_name" key
"org_name" api
"org_name" "filename:vim_settings.xml"
"org_name" "Authorization: Bearer"
"org_name" "Language: PHP"
```

### Useful GitHub search qualifiers (build your own dorks)

```
user:name          # e.g. user:admin — search a specific account
org:name           # e.g. org:google type:users — org members
in:login           # <username> in:login
in:name            # <username> in:name
fullname:<first> <last>
in:email           # data in:email
created:<2012-04-05
created:>=2011-06-12
created:2016-02-07 location:iceland
created:2011-04-06..2013-01-14 <user> in:username
extension:pem private
extension:ppk private
extension:sql mysql dump
extension:json api.forecast.io
```

The full grouped dork lists (files, sensitive files, language-based, API keys/tokens/passwords, usernames, dates, extensions) are maintained in **[Github-dorks.md](../Sensitive_Info_Leaks/Github-dorks.md)** and **[Github_dorks_all.md](../Sensitive_Info_Leaks/Github_dorks_all.md)**.

## Bypass Techniques

- GitHub code search throttles unauthenticated queries — use a logged-in account you don't mind risking, and space out queries.
- Leaks hide in **gists and forks**, which default searches miss; grep candidate users' gists directly.
- Deleted/edited secrets often survive in **commit history** — a secret scanner (below) reads full histories, unlike web search.

## Tools & Resources

1. [TruffleHog](https://github.com/dxa4481/truffleHog) — deep credential scanning of repos (full commit history)
2. [WatchTower](https://radar.nightfall.ai/) — continuous leak monitoring
3. [Dorki](https://dorki.attaxa.com/) — GitHub dork automation
4. [GitGot](https://github.com/BishopFox/GitGot), [gitGraber](https://github.com/hisxo/gitGraber), [git-hound](https://github.com/tillson/git-hound) — org/employee-targeted dork runners (see [Github_Recon_Method.md](../Sensitive_Info_Leaks/Github_Recon_Method.md))

## Report Tips

- **NOTE:** If you find any API key, credentials, or other sensitive information under a test directory, do not report it — that is intended behavior.
- Prove impact minimally: validate the token with a read-only/safe call (e.g. identify the owning account), never exfiltrate third-party data or access production.
- Include the exact commit/blob URL and timestamp; GitHub is public evidence triage can verify instantly.

## Author

[Mr._fr3qu3n533](https://twitter.com/mr_fr3qu3n533)
