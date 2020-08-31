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

```
### References
[Medium Writeup](https://medium.com/swlh/intro-to-csrf-cross-site-request-forgery-9de669df03de)

[Medium Writeup](https://medium.com/swlh/attacking-sites-using-csrf-ba79b45b6efe)

[Medium Writeup](https://medium.com/swlh/bypassing-csrf-protection-c9b217175ee)

### Author
**Name:** Syed Mushfik Hasan Tahsin

**Handle:** [@SMHTahsin33](https://twitter.com/SMHTahsin33)
