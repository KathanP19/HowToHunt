# GitHub Dorks — Curated Essentials

A hand-picked set of GitHub code-search dorks that surface the highest-hit-rate leaks: credential/config files, API keys, tokens, private keys, and dumps. Start here; the exhaustive corpus (thousands of secret-keyword and regex patterns) lives in [Github_dorks_all.md](Github_dorks_all.md), which deliberately contains **only** what is not listed below — together the two files are the complete collection without duplication.

**Why it matters / typical impact:** these dorks find working credentials — cloud keys, DB passwords, SSH keys, service tokens — which frequently yield direct account or infrastructure takeover. For the full hunting workflow see [Recon/Github_Dorking.md](../Recon/Github_Dorking.md) and [Github_Recon_Method.md](Github_Recon_Method.md).

## Hunting Checklist

1. Pick a target brand/product/org name and prefix the file/extension dorks below with `"org_name"` (patterns in [Recon/Github_Dorking.md](../Recon/Github_Dorking.md)).
2. Run the API-key/token keyword dorks scoped by `org:` and `user:` for employee accounts.
3. Check hit recency and whether the secret is still valid *safely* (read-only validation) before going further.
4. For automated sweeps, graduate to [Github_dorks_all.md](Github_dorks_all.md) + a runner tool (below).

## Dork Lists

### GitHub dorks for finding files

```
filename:manifest.xml
filename:travis.yml
filename:vim_settings.xml
filename:database
filename:prod.exs NOT prod.secret.exs
filename:prod.secret.exs
filename:.npmrc _auth
filename:.dockercfg auth
filename:WebServers.xml
filename:.bash_history <Domain name>
filename:sftp-config.json
filename:sftp.json path:.vscode
filename:secrets.yml password
filename:.esmtprc password
filename:passwd path:etc
filename:dbeaver-data-sources.xml
path:sites databases password
filename:config.php dbpasswd
filename:configuration.php JConfig password
filename:.sh_history
shodan_api_key language:python
filename:shadow path:etc
JEKYLL_GITHUB_TOKEN
filename:proftpdpasswd
filename:.pgpass
filename:idea14.key
filename:hub oauth_token
HEROKU_API_KEY language:json
HEROKU_API_KEY language:shell
SF_USERNAME salesforce
filename:.bash_profile aws
filename:.env MAIL_HOST=smtp.gmail.com
filename:wp-config.php
filename:credentials aws_access_key_id
filename:id_rsa or filename:id_dsa
```

### GitHub dorks for finding languages

```
language:python username
language:php username
language:sql username
language:html password
language:perl password
language:shell username
language:java api
HOMEBREW_GITHUB_API_TOKEN language:shell
```

### API keys, tokens & hard-coded passwords

```
api_key
SecretKey / Secrect_key / skey
privatekey / private_key / pkey
user_secret / userSecret
admin_passwd / adminpasswd / adminPass etc
"api keys"
authorization_bearer:
oauth
auth
authentication
client_secret
api_token:
"api token"
client_id
password
user_password
user_pass
passcode
secret
password hash
OTP
user auth
```

### Username & date qualifiers

```
user:name                     (user:admin)
org:name                      (org:google type:users)
in:login                      (<username> in:login)
in:name                       (<username> in:name)
fullname:firstname lastname   (fullname:<name> <surname>)
in:email                      (data in:email)

created:<2012-04-05
created:>=2011-06-12
created:2016-02-07 location:iceland
created:2011-04-06..2013-01-14 <user> in:username
```

### Extension-based dorks

```
extension:pem private
extension:ppk private
extension:sql mysql dump
extension:sql mysql dump password
extension:json api.forecast.io
extension:json mongolab.com
extension:yaml mongolab.com
[WFClient] Password= extension:ica
extension:avastlic "support.avast.com"
extension:json googleusercontent client_secret
```

## Tools & Resources

- [GitHub code search](https://github.com/search?q=&type=code) — run dorks manually (requires login for code search)
- [TruffleHog](https://github.com/dxa4481/truffleHog) — automated secret scanning of repos/history
- [gitGraber](https://github.com/hisxo/gitGraber), [GitGot](https://github.com/BishopFox/GitGot), [git-hound](https://github.com/tillson/git-hound) — org/employee-targeted dork runners
- [SecurityTrails GitHub dorks guide](https://securitytrails.com/blog/github-dorks)
- Exhaustive pattern corpus: [Github_dorks_all.md](Github_dorks_all.md)

## Report Tips

- Validate the secret minimally and read-only (e.g. identify the owning account/service); never pivot into third-party systems — many keys belong to vendors, not the program.
- Report the exact repo/blob/commit URL with timestamp and what the credential grants access to; that framing turns "info leak" into critical.
- Exclude anything under test/example directories — those are intended behavior, not bugs.
