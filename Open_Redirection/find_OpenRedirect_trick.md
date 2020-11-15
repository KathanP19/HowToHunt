## A small trick to find Open Redirection if you couldn't find any Redirection parameters.

*"I apply this everytime while testing web applications and found many Open Redirects and even an XSS using this trick!"*

### Steps:
------------------------------------------------------------------------------------------------------------------------------------------------------------
      1. If the Applictaion have a user Sign-In/Sign-Up feature, then register a user and log in as the user.
      
      2. Go to your user profile page , for example : samplesite.me/accounts/profile
      
      3. Copy the profile page's URL
      
      4. Logout and Clear all the cookies and go to the homepage of the site.
      
      5. Paste the Copied Profile URL on the address bar 
      
      6. If the site prompts for a login , check the address bar , you may find the login page with a redirect parameter like the following
            - https://samplesite.me/login?next=accounts/profile
            - https://samplesite.me/login?retUrl=accounts/profile
      
      7. Try to exploit the parameter by adding an external domain and load the crafted URL
          eg:- https://samplesite.me/login?next=https://evil.com/
                         (or)
            https://samplesite.me/login?next=https://samplesite.me@evil.com/  #(to beat the bad regex filter)
      
      8. If it redirects to evil.com , thers's your open redirection bug.
       
      9. Try to leverage it to XSS
           eg:- https://samplesite.me/login?next=javascript:alert(1);//

-------------------------------------------------------------------------------------------------------------------------------------------------------------
       
 #### Author:  [febinrev](https://twitter.com/febinrev) 
