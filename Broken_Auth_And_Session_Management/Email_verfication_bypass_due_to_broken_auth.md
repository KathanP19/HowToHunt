
## Broken Authentication To Email Verification Bypass (P4) :

------------------------------------------------------------
------------------------------------------------------------

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

------------------------------------------------------------------------------------
------------------------------------------------------------------------------------

`Happy hacking :`

### Author :

* Linkedin : [@chirag_Agrawal](https://www.linkedin.com/in/chirag-agrawal-770488144/)
* Twitter  : [@Raiders](https://twitter.com/ChiragA15977205)



