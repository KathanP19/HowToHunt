<h4>Summary:</h4>

A weak password policy increases the probability of an attacker having success using brute force and dictionary attacks against user accounts. An attacker who can determine user passwords can take over a user's account and potentially access sensitive data in the application.

<h4>Steps to reproduce:</h4>

1. Create a new account and use the email address as the password. </br>
2. Reset your password and choose your email address as the password. </br>
   In both cases, the application does not prevent this decision. </br>

To improve the password strength, the application should avoid 1-to-1 usage of personal information as the account password.

Author: [@0xd3vil](https://twitter.com/0xd3vil)
