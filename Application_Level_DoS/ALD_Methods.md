
## 1. Email Bounce Issues
- Check if Application has Invite Functionality
- Try sending Invites to Invalid Email Accounts
- Try to find Email Service Provider such as AWS SES , Hubspot , Campaign Monitor
**Note:  You can find Email Service Provider by checking Email Headers**
* Once you have the Email Service Provider, Check there Hard Bounce Limits. Here are the limits for some of them:
  
  **1. Hubspot Hard bounces:** HubSpot's hard bounce limit is 5%. For reference, many ISPs prefer bounce rates to be under 2%.
  
  **2. AWS SES:** The rate of SES ranges from first 2-5% then 5-10%

***Impact: Once the Hard Bounce Limits are reached, Email Service Provider will block the Company which means, No Emails would be sent to the Users !***

## 2. Long Password DoS Attack

- As the value of password is hashed and then stored in Databases. If there is no limit on the length of the Password, it can lead to consumption of resources for Hashing the Long Password.

**How to test?**

- Use a Password of length around 150-200 words to check the presense of Length Restriction
- If there is no Restriction, Choose a longer password and keep a eye on Response Time
- Check if the Application Crashes for few seconds 

**Where to test?**

- Registration Password Field is usually restricted but the Length of Password on the Forgot Password Page and the Change Password (As Authenticated User) Functionality is usually missing.


## 3. Long String DOS

* When you set some string so long so server cannot process it anymore it cause DOS sometime

**How to test**
```
Create app and put field like username or address or even profile picture name parameter ( second refrence ) like 1000 character of string . 
Search A's account from B's account either it will
```
- Either it will keeping on searching for long time
- Either the application will crash (500 - Error Code)

## Reference : 
\- Email Bounce Issues
* [https://medium.com/bugbountywriteup/an-unexpected-bounty-email-bounce-issues-b9f24a35eb68](https://medium.com/bugbountywriteup/an-unexpected-bounty-email-bounce-issues-b9f24a35eb68)

\- Long Password DoS Attack

- https://www.acunetix.com/vulnerabilities/web/long-password-denial-of-service/
- https://hackerone.com/reports/738569
- https://hackerone.com/reports/167351

\- Long String DOS
- [https://medium.com/@shahjerry33/long-string-dos-6ba8ceab3aa0](https://medium.com/@shahjerry33/long-string-dos-6ba8ceab3aa0)
- https://hackerone.com/reports/764434

## Author: 
* [Keshav Malik](https://twitter.com/g0t_rOoT_)
