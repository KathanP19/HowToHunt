# CSRF

Cross-Site Request Forgery (CSRF) tricks a logged-in victim's browser into submitting state-changing requests (with their cookies attached) that the site accepts as legitimate — up to account takeover via email/password change, transfers, or privilege actions.

This folder:

- [CSRF.md](CSRF.md) — core methodology: preconditions, Burp PoC generation, worked token/Referer/CRLF bypass test cases, mitigations.
- [Cross_Site_Request_Forgery_Bypass.md](Cross_Site_Request_Forgery_Bypass.md) — quick-fire bypass cheat list (blank/null token, same-length swap, content-type switch, Referer regex, clickjacking).

## MindMaps

### 6 CSRF Bypass by Hack3rSr0lls
![https://pbs.twimg.com/media/EY70bxkWkAAFzGb?format=jpg&name=900x900](https://pbs.twimg.com/media/EY70bxkWkAAFzGb?format=jpg&name=900x900)

### CSRF Mindmap
![https://gblobscdn.gitbook.com/assets%2F-L_2uGJGU7AVNRcqRvEi%2F-LrAtLpniLVMCWL-CVF-%2F-LrAtNhzv9bhi7vB_zs7%2Fimage.png?alt=media&token=ead94d04-f31d-4d99-9087-9bf92d091b0b](https://gblobscdn.gitbook.com/assets%2F-L_2uGJGU7AVNRcqRvEi%2F-LrAtLpniLVMCWL-CVF-%2F-LrAtNhzv9bhi7vB_zs7%2Fimage.png?alt=media&token=ead94d04-f31d-4d99-9087-9bf92d091b0b)

### Source
* [https://twitter.com/hackerscrolls/status/1265217322308046849](https://twitter.com/hackerscrolls/status/1265217322308046849)

### Author
* [KathanP19](https://twitter.com/KathanP19)
