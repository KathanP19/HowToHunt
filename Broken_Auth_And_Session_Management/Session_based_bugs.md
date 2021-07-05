### Old Session Does Not Expire:
* Steps:
```
      1.create your account
      2.open two browser eg.,chrome and firefox
      3.Login in one browser eg.chrome
      4.In other browser(firefox) login either change your password or reset your password
      5.After successfully changed or reset go to other browser refresh the page if you are still logged in
```      
Than this is an old session does not expire bug
### Session Hijacking (Intended Behavior)
* Steps:
```
    1.Create your account
    2.Login your account
    3.Use cookie editor extension in browser
    4.Copy all the target cookies
    5.Logout your account
    6.Paste that cookies in cookie editor extension
    7.Refresh page if you are logged in than this is a session hijacking
```  
`Impact:` If attacker get cookies of victim it will leads to account takeover.
 
 
### Password reset link token does not expire (Insecure Configurability)
* Steps:
```
      1.Create your account on target
      2.request a forget password link
      3.Don't use that link
      4.Instead logged in with your old password and change your email to other
      5.Now use that password link sents to old email and check if you are able to change your password if yes than there is the title bug.
 ```    
 
 ### Server security misconfiguration -> Lack of security headers -> Cache control for a security page
 * Steps :
 ``` 
     1. Login to the application
     2. Navigate around the pages
     3. Logout
     4. Press (Alt+left-arrow) buttons
     5. If you are logged in or can view the pages navigated by the user. Then you found a bug.
  ```
  `Impact:` At a PC cafe, if a person was in a very important page with alot of details and logged out, then another person comes and clicks back (because he didnt close the browser) then data is exposed. User information leaked
 
 ### Broken Authentication To Email Verification Bypass (P4) :
  `category` : P4 >> Broken Authentication and Session Management >> Failure to Invalidate Session >> On Password Reset and/or Change

* Steps To Reproduce:
``` 
    1)First You need to make a account & You will receive a Email verification link.
    2)Application in my case give less Privileges & Features to access if not verified.
    3)Logged into the Application & I change the email Address to Email B.
    4)A Verification Link was Send & I verified that.
    5) Now I again Changed the email back to Email I have entered at the time of account creation.
    6) It showed me that my Email is Verified.
    7) Hence , A Succesful Email verfication Bypassed as I haven't Verified the Link which was sent to me in the time of account creation still my email got verified.
    8)Didn't Receive any code again for verification when I changed back my email & When I open the account it showed in my Profile that its Verified Email.
```

`Impact` :
Email Verfication was bypassed due to Broken Authentication Mechanism , Thus more Privileged account can be accessed by an attacker making website prone to Future Attacks.    
  Happy Hacking:)
  
  ## Resources:
  Google,Youtube.

## Authors
* [https://twitter.com/Virdoex_hunter](https://twitter.com/Virdoex_hunter)
* Linkedin : [@chirag_Agrawal](https://www.linkedin.com/in/chirag-agrawal-770488144/), Twitter  : [@Raiders](https://twitter.com/ChiragA15977205)
