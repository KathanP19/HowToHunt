### Application Level DoS

- Check if Application has Invite Functionality
- Try sending Invites to Invalid Email Accounts
- Try to find Email Service Provider such as AWS SES , Hubspot , Campaign Monitor

`Note  You can find Email Service Provider by checking Email Headers`

Once you have the Email Service Provider, Check there Hard Bounce Limits. Here are the limits for some of them:

- Hubspot Hard bounces: HubSpot's hard bounce limit is 5%. For reference, many ISPs prefer bounce rates to be under 2%.
- AWS SES: The rate of SES ranges from first 2-5% then 5-10%

***Impact: Once the Hard Bounce Limits are reached, Email Service Provider will block the Company which means, No Emails would be sent to the Users !***


### Reference : 
* [https://medium.com/bugbountywriteup/an-unexpected-bounty-email-bounce-issues-b9f24a35eb68](https://medium.com/bugbountywriteup/an-unexpected-bounty-email-bounce-issues-b9f24a35eb68)

### Author: 
* (Keshav Malik)[twitter.com/g0t_rOoT_]
