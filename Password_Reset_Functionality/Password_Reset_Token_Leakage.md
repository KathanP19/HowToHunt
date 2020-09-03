Password Reset Token Leakage
    
    Steps:
    1.Sent a password reset request using forget password
	2.Check your email 
	3.copy your reset page link and paste in another tab and make burp intercept on.
	4.Look for every request if you find similiar token that is in reset link with other domain like: bat.bing.com or facebook.com
	5.Than there is reset password token leakage.

### Authors

* [@Virdoex_hunter](https://twitter.com/Virdoex_hunter)
