## Flaw-Name : Unlimited Email Triggering
---
### Steps To Reproduce :
- 1 - Navigate to : `https://abc.target.com/verify-email`
- 2 - `Intercept` the request in BurpSuite
- 3 - Send the request to `Intruder` and clear the payload position
- 4 - Use `Null payloads` as payload type and set the payload count to 100
- 5 - `Start attack`

---
### Impact : 
- If the company is using any email service software API or some tool that has been bought for the emails being sent on the support domain, the rate limit can result in `financial loss` and it can also `slow down your services` as huge/mass mails will `lead to disruption of data` that original user might send or the quota that has been bought might be exhausted.
---
### Mitigation :

- 1 - IP Based Blocking
- 2 - Captcha
- 3 - Firewall
- 4 - Reducing the number of API requests
