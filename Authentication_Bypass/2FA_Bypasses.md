# 2FA Bypass Techniques

### [Mindmap](https://mm.tt/1736437018?t=SEeZOmvt01)

Index | Technique
--- | ---
**1** | Response Manipulation
**2** | Status Code Manipulation
**3** | 2FA Code Leakage in Response
**4** | JS File Analysis
**5** | 2FA Code Reusability
**6** | Lack of Brute-Force Protection
**7** | Missing 2FA Code Integrity Validation
**8** | CSRF on 2FA Disabling
**9** | Password Reset Disable 2FA
**10** | Backup Code Abuse
**11** | Clickjacking on 2FA Disabling Page
**12** | Enabling 2FA doesn't expire Previously active Sessions
**13** | Bypass 2FA with null or 000000
___
#### Response Manipulation
```
In response if "success":false
Change it to "success":true
```

#### Status Code Manipulation

```
If Status Code is 4xx
Try to change it to 200 OK and see if it bypass restrictions
```
#### 2FA Code Leakage in Response
```
Check the response of the 2FA Code Triggering Request to see if the code is leaked.
```
#### JS File Analysis
```
Rare but some JS Files may contain info about the 2FA Code, worth giving a shot
```
#### 2FA Code Reusability
```
Same code can be reused
```
### Lack of Brute-Force Protection
```
Possible to brute-force any length 2FA Code
```
#### Missing 2FA Code Integrity Validation
```
Code for any user acc can be used to bypass the 2FA
```
#### CSRF on 2FA Disabling
```
No CSRF Protection on disabling 2FA, also there is no auth confirmation
```
#### Password Reset Disable 2FA
```
2FA gets disabled on password change/email change
```
#### Backup Code Abuse
```
Bypassing 2FA by abusing the Backup code feature
Use the above mentioned techniques to bypass Backup Code to remove/reset 2FA restrictions
```
#### Clickjacking on 2FA Disabling Page
```
Iframing the 2FA Disabling page and social engineering victim to disable the 2FA
```
#### Enabling 2FA doesn't expire Previously active Sessions
```
If the session is already hijacked and there is a session timeout vuln
```
#### Bypass 2FA with null or 000000
```
Enter the code 000000 or null to bypass 2FA protection.
```
___

### Articles 
- [Testing Two-Factor Authentication](https://research.nccgroup.com/2021/06/10/testing-two-factor-authentication/) by [NCC Group](https://research.nccgroup.com/)


## Author

[Harsh Bothra](https://twitter.com/harshbothra_) \
[Vishal Saini](https://twitter.com/k4k4r07)
