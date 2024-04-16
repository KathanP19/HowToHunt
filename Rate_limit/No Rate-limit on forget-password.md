## Flaw-Name : No rate limit on forget/reset password leads to email triggering
---
### Steps To Reproduce 
- 1 - Navigate to : `https://abc.target.com/forgot-password` or it could be `https://abc.target.com/reset-password`
- 2 - Enter the email of the victim
- 3 - `Intercept` the request in burp suite
- 4 - Send the request to the `Intruder` and clear payload positions
- 5 - Use `Null payloads` and set the payload count to 100
- 6 - `Start attack`


---
### Impact : 
- If the company is using any email service software API or some tool that has been bought for the emails being sent on the support domain, the rate limit can result in `financial loss` and it can also `slow down your services` as huge/mass mails will `lead to disruption of data` that original user might send or the quota that has been bought might be exhausted.
---
### Mitigation :

- 1 - IP Based Blocking
- 2 - Captcha
- 3 - Firewall
- 4 - Reducing the number of API requests