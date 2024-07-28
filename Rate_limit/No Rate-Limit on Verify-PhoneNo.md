## Flaw-Name : Unlimited SMS Triggering
---
### Steps To Reproduce 
- 1 - Open this url `https://target.com/phone-number-verify`
- 2 - Enter the `victim's mob. number`
- 3 - `Intercept the request` and send the request to intruder
- 4 - Use payload type as `NULL payloads` and set the payload count & `start attack`
---
### Impact : 
- If the company is using any email service software API(such as AWS,GCP..etc) or some tool that has been bought for the emails being sent on the support domain, the rate limit can result in `financial loss` and it can also `slow down your services` as huge/mass mails will `lead to disruption of data` that original user might send or the quota that has been bought might be exhausted.
---
### Mitigation - 

- 1 - IP Based Blocking
- 2 - Captcha
- 3 - Firewall
- 4 - Reducing the number of API requests
