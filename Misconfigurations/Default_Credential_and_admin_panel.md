# Default Credentials Basics

- Default Software Configurations for admin console of webapp
```
https://www.target.com/admin
https://www.target.com/admin-console
https://www.target.com/console
https://admin.target.com
https://admin-console.target.com
https://console.target.com
```

# Getting access through third party services

* When the admin console login page is working on a third party service,then just search for it's default credentials on Google
* Third Party service URL are of the format: https://target.<TPS Name>.com/login
* Some examples are Okta,WP etc

# Bypass to get access to login page

1. This bypass is used when you are forbidden to get access to admin login page

2. We use Header Injection for this bypass

3. `X-Orginal-URL: /admin` or `X-Rewrite-URL:/admin`

4. Use this Header under Host

* Use Burp to capture then check

# Hackerone Reports :
- https://hackerone.com/reports/192074
- https://hackerone.com/reports/174883
- https://hackerone.com/reports/398797

# Source : 
https://www.owasp.org/index.php/Testing_for_default_credentials_(OTG-AUTHN-002)
https://www.owasp.org/index.php/Testing_for_Default_or_Guessable_User_Account_(OWASP-AT-003)

## Author:
* [@e11i0t_4lders0n](https://twitter.com/e11i0t_4lders0n)


