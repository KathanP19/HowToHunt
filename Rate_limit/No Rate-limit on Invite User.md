## Flaw-Name : No rate limit on invite user leads to email triggering
---

### Description : 
- Rate limiting is a strategy for limiting network traffic. It puts a cap on how often someone can repeat an action within a certain timeframe – for instance, trying to log in to an account.

---
### Steps To Reproduce 
- 1 - Go to `https://target.com/`
- 2 - Navigate to `Invite User` option and Enter the `victim's email` 
- 3 - Send invite & `Intercept` the Request
- 4 - Send the request to `Intruder` & clear payload positions
- 5 - Apply payload type as `null payload` and payload count as 100
- 5 - Click on `Start attack` after applying the threads
- 6 - The victim will get huge number of emails
---
### Impact : 
- If the company is using any email service software API(such as AWS,GCP..etc) or some tool that has been bought for the emails being sent on the support domain, the rate limit can result in `financial loss` and it can also `slow down your services` as huge/mass mails will `lead to disruption of data` that original user might send or the quota that has been bought might be exhausted.
---
### Mitigation - 

- 1 - IP Based Blocking
- 2 - Captcha
- 3 - Firewall
- 4 - Reducing the number of API requests
