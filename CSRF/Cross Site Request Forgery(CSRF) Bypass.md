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
Author: [@SMHTahsin33](https://twitter.com/SMHTahsin33)
