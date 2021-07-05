
## Summary
Password reset links are usually addressed to your account name followed by the reset link. Also if the application allows
you to have your account name with tags and special characters then you should try this.

### Steps

1. Create your account
2. Edit your name to `<h1>attacker</h1>` or `"abc><h1>attacker</h1>` and save it.
3. Request for a reset password and check your email.
4. You will notice the `<h1>` tag getting executed

* HTML injection are usually considered as low to medium severity bugs but you can escalate the severity by serving a 
malicious link by using `<a href>` for eg: `<h1>attacker</h1><a href="your-controlled-domain"Click here</a>`

* You can redirect the user to your malicious domain and serve a fake reset password page to steal credentials 
Also you can serve a previously found XSS page and steal user cookies etc etc.. The creativity lies on you..

## Author
[@C1pher15](https://twitter.com/C1pher15)
