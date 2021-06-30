
## Chaining Session Hijacking with XSS
```
1.I have add a session hijacking method in broken auth and session managment. 
2.If you find that on target.
3.Try anyway to steal cookies on that target.
4.Here I am saying look for xss .
5.If you find xss you can stole the cookies of victim and using session hijacking you can takeover the account of victim.
```
##  No Rate Limit On Login With Weak Password Policy
```
So if you find that target have weak password policy try to go for no rate limit attacks in poc shows by creating very weak password of your account.

(May or may not be accepted)
```
## Password Reset Poisioning Leads To Token Theft
```
1.Go to password reset funtion.
2.Enter email and intercept the request.
3.Change host header to some other host i.e,
    Host:target.com
    Host:attacker.com
  also try to add some headers without changing host like
    X-Forwarded-Host: evil.com
    Referrer: https://evil.com
4.Forward this if you found that in next request attacker.com means you successfully theft the token.:)
```
## Using  Auth Bypass
```
Check out Auth Bypass method, there is a method for otp bypass via response manipulation this can leads to account takeovers.
```
## Try For CSRF On
```
1.Change Password function.
2.Email change
3.Change Security Question
```
## Token Leaks In Response

* So there are multiple ways to do it but all are same.

* So I will sharing my method that I have learnt here .

* Endpoints:(Register,Forget Password)

* Steps(For Registration):
```
  1.for registeration intercept the signup request that contains data you have entered.
  2.Click on action -> do -> intercept response to this request.
  3.Click forward.
  4.Check response it that contains any link,any token or otp.
 ```
 ------------------------
 * Steps(For password reset):
 ``` 
  1.Intercept the forget password option.
  2.Click on action -> do -> intercept response to this request.
  3.Click forward.
  4.Check response it that contains any link,any token or otp.
 ```

## Reference:
* Various Source From Google,Twitter,Medium

## Author
* [@Virdoex_hunter](https://twitter.com/Virdoex_hunter)
