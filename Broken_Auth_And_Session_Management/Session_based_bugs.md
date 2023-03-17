# Broken Authentication And Session Management.

### Old Session Does Not Expire After Password Change:
* Steps:
```
      1.create An account On Your Target Site
      2.Login Into Two Browser With Same Account(Chrome, FireFox.You Can Use Incognito Mode As well) 
      3.Change You Password In Chrome, On Seccessfull Password Change Referesh Your Logged in Account In FireFox/Incognito Mode.
      4.If you'r still logged in Then This Is a Bug
```      

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
 
 
### Password reset token does not expire (Insecure Configurability)
* Steps:
```
      1.Create your account on target Site.
      2.request for a forget password token.
      3.Don't use that link
      4.Instead logged in with your old password and change your email to other.
      5.Now use that password link sents to old email and check if you are able to change your password if yes than there is the litle bug.
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
  
  ### Email Verification Bypass (P3/P4)
  * Steps :
   ``` 
    1)First You need to Create an account with Your Own Email Address.
    2)After Creating An Account A Verification Link will be sent to your account.
    3)Dont Use The Email Verification link. Change Your Email to Victim's Email.
    4)Now Go in Your Email and Click on Your Own Email Verification Link.
    5)if the Victim's Email Get Verified then This is a Bug.
```
`Impact` : Email Verfication Bypass

 ### Old Password Reset Token Not Expiring Upon Requesting New One (Sometimes P4) :
  * Steps :
 ``` 
    1)First You need to Create an account with a Valid Email Address.
    2)After Creating An Account log out from your Account and Navigate on Forgot Password Page.
    3)Request a Password Reset Link for your Account.A Verification Link will be sent to your account.
    4)Without Using this Password Reset Link Request A New Password Reset Link.
    5)Now go in Your email and Use 1st Password Reset Link Rather than Using 2nd One And Change Your Password.
    6) If You Are Able to Change Your Password Than This Is a tiny Bug ;).
```
* Note:- Some Companies Won't Accept it As Valid Issue. 

### Password Reset Token Not Expiring After Password Change (P4):
  * Steps :
 ``` 
    1)First You need to Create an account with a Valid Email Address.
    2)After Creating An Account log out from your Account and Navigate on Forgot Password Page.
    3)Request a Password Reset Link for your Account.
    4)Use The Password Reset Link And Change The Password, After Changing the Password Login to Your Account.
    5)Now Use The Old Password Reset Link To Change The Password Again.
    6) If You Are Able to Change Your Password Again Than This Is a tiny Bug  ;).
```

### Insufficient account process validation leads to account takeover (P3/P4):
   * Steps :
```
      1) Create an account on the website.
      2) Go to profile section. And Change & update your details in the name parameter and before saving it Open Burp suite, turn the proxy on and then click on Save.
      3) Now capture the request in Burp suite and send it to the Repeater tab.
      4) Now log out from the website and go back to the Burp suite.
      5) Now change the details email & name parameters and click on "Go" in the repeater tab.
      6) Now you will be able to see 200 ok response from the web server.
      7) Now, login into your account and go to the Profile section to confirm
```

* Thanks For Reading Guys Happy Hunting :).

  ## Resources:
  Google,Youtube.

## Authors
* [https://twitter.com/Virdoex_hunter](https://twitter.com/Virdoex_hunter) 
* Linkedin : [@chirag_Agrawal](https://www.linkedin.com/in/chirag-agrawal-770488144/), Twitter  : [@Raiders](https://twitter.com/ChiragA15977205)
* Twitter : [Fani Malik](https://twitter.com/fanimalikhack) 
* Linkedin : [@suprit-pandurangi](https://www.linkedin.com/in/suprit-pandurangi-a90526106/)
