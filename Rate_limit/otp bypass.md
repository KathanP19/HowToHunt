## Flaw-Name : OTP Bypass
---
### Steps To Reproduce :
- 1 - Go to the URL : `https://abc.target.com/verify/phoneno`
- 2 - Enter random code (000000)
- 3 - `Intercept` the Request
- 4 - Send it to the `Intrude`r and add a payload position on `OTP parameter`
- 5 - Apply payload type as numbers, Set the range(000000-999999) and step as 1
- 6 - Start attack
- 7 The response length will change at correct OTP
---
### Impact : 
- The attacker will be able to bypass the OTP which can lead to an Zero Account Takeover
---
### Mitigation :

- 1 - IP Based Blocking
- 2 - Captcha
- 3 - Firewall
- 4 - Reducing the number of API requests
