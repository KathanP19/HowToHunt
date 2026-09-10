# GitHub Recon Method

GitHub is one of the highest-yield sources of sensitive information leaks: company names, employee accounts, and source-code references all lead back to repositories where secrets get committed. A structured method — org, people, then code references — finds leaks that random dorking misses. Typical impact: exposed tokens/API keys, internal service details, and build/deploy credentials usable for account or infrastructure takeover.

## Hunting Checklist

1. **Search GitHub by company name** for API keys, passwords, tokens, and internal configs (org-scoped patterns in [Recon/Github_Dorking.md](../Recon/Github_Dorking.md)).
2. **Enumerate employees** from LinkedIn/Twitter (and repo contributor lists), then check their personal repositories and gists for sensitive information tied to the target.
3. **Mine the website itself** — check the source of the main site and subdomains for GitHub links in HTML comments or anywhere else; use Ctrl+F in view-source and search for the keyword `github`.
4. **Work the dork lists** — curated: [Github-dorks.md](Github-dorks.md); exhaustive automation corpus: [Github_dorks_all.md](Github_dorks_all.md).
5. **Validate safely & report** — read-only verification only, then follow the report tips below.

## Test Cases & Examples

```bash
# Manual search entry points
# https://github.com/search?q=<company_name>+password&type=code
# https://github.com/search?q=org%3A<org>+filename%3A.env&type=code

# Pull a page's HTML and hunt for github references (comments, source maps, JS bundles)
curl -sk https://target.com | grep -io 'github[^"]*' | sort -u
```

Dork corpora: run the patterns from [Github-dorks.md](Github-dorks.md) and [Github_dorks_all.md](Github_dorks_all.md) through GitHub search or one of the tools below.

## Tools & Resources

- [GitGot](https://github.com/BishopFox/GitGot) — search GitHub for orgs/repos and clone what it finds
- [gitGraber](https://github.com/hisxo/gitGraber) — GitHub dorking for organizations/employees
- [git-hound](https://github.com/tillson/git-hound) — targeted subdomain + GitHub info discovery
- [SecurityTrails: GitHub dorks guide](https://securitytrails.com/blog/github-dorks)
- Dork lists: [Github-dorks.md](Github-dorks.md) · [Github_dorks_all.md](Github_dorks_all.md)

## Reports (HackerOne)

### Resolved

- [Important information leaked on Github](https://hackerone.com/reports/649322)
- [Github Token Leaked publicly for https://github.com/mopub](https://hackerone.com/reports/612231)
- [CircleCI token in github repo allows for access to sensitive build information](https://hackerone.com/reports/858915)
- [Information Leak - Github - JMS Information](https://hackerone.com/reports/360811)
- [Leaked artifactory_key, artifactory_api_key, and gcloud refresh_token via GitHub.](https://hackerone.com/reports/496414)
- [Github Token Leaked publicly for https://github.sc-corp.net](https://hackerone.com/reports/396467)

## Report Tips

- Show what the leaked credential actually grants access to (build systems, internal registries, cloud accounts) — that's the difference between informative and critical in the reports above.
- Include exact repo/blob URLs and timestamps; state that validation was read-only.
- Do **not** report secrets sitting in test/example directories or clearly fake placeholder values — intended behavior.

## Author

- [@0xCCFFF](https://twitter.com/0xCCFFF) (MadMaxx)
- [@klaus](https://twitter.com/klaus_dev)
