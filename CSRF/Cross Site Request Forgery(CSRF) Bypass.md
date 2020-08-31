**Cross Site Request Forgery(CSRF)**

Hello Guys, I Tried My Best To Share all The CSRF Bypasses I Know.
I Hope This Will Help You.

```
-Change Request Method [POST => GET]

-Remove Total Token Parameter

-Remove The Token, And Give a Blank Parameter

-Copy a Unused Valid Token , By Dropping The Request and Use That Token

-Use Own CSRF Token To Feed it to Victim

-Replace Value With Of A Token of Same Length 

-Reverse Engineer The Token

-Extract Token via HTML injection

-Switch From Non-Form `Content-Type: application/json` or `Content-Type: application/x-url-encoded` To `Content-Type: form-multipart`

-Bypass the regex
  If the site is looking for “bank.com” in the referer URL, maybe “bank.com.attacker.com” or “attacker.com/bank.com” will work.
    
-Remove the referer header (add this <meta name=”referrer” content=”no-referrer”> in your payload or html code)

-Clickjacking

  (If you aren’t familiar with clickjacking attacks, more information can be found https://owasp.org/www-community/attacks/Clickjacking.)
  Exploiting clickjacking on the same endpoint bypasses all CSRF protection. Because technically, the request is indeed originating from the legitimate site. If the page where   the vulnerable endpoint is located on is vulnerable to clickjacking, all CSRF protection will be rendered irrelevant and you will be able to achieve the same results as a CSRF   attack on the endpoint, albeit with a bit more effort.
	


```

### References
[Medium Writeup](https://medium.com/swlh/intro-to-csrf-cross-site-request-forgery-9de669df03de)

[Medium Writeup](https://medium.com/swlh/attacking-sites-using-csrf-ba79b45b6efe)

[Medium Writeup](https://medium.com/swlh/bypassing-csrf-protection-c9b217175ee)


# Authors

* [@SMHTahsin33](https://twitter.com/SMHTahsin33)
* [@Virdoex_hunter](https://twitter.com/Virdoex_hunter)
* [@remonsec](https://twitter.com/remonsec)

