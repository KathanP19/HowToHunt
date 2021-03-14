# Github Recon
Using Github we can find sensitive infos.

## Steps:

1. Check github with company name for API keys or passswords.
2. Enumerate the employees of the company from linkedin and twitter and check their repositories on github for sensitive information.
3. Check source code of main website and subdomains for github links in the html comments or anywhere. Search using ctl-F and search for keyword github

## Tools and references::
* https://github.com/BishopFox/GitGot
* https://github.com/hisxo/gitGraber
* https://github.com/tillson/git-hound
* https://securitytrails.com/blog/github-dorks

## Reports (Hackerone)

### Resolved

- [Important information leaked on Github](https://hackerone.com/reports/649322)
- [Github Token Leaked publicly for https://github.com/mopub](https://hackerone.com/reports/612231)
- [CircleCI token in github repo allows for access to sensitive build information](https://hackerone.com/reports/858915)
- [Information Leak - Github - JMS Information](https://hackerone.com/reports/360811)
- [Leaked artifactory_key, artifactory_api_key, and gcloud refresh_token via GitHub.](https://hackerone.com/reports/496414)
- [Github Token Leaked publicly for https://github.sc-corp.net](https://hackerone.com/reports/396467)

## Author:
* [@0xCCFFF](https://twitter.com/0xCCFFF) (MadMaxx)
* [@klaus](https://twitter.com/klaus_dev)
