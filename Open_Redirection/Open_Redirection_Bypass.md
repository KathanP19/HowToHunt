# Open Redirection Bypass Trick:

This bypass I found in a application while I doing pentesting. I hope it will helps you too!

1. While you I trying to redirect https://targetweb.com?url=http://attackersite.com it did not redirected!
2. I Created a new subdomain with with www.targetweb.com.attackersite.com
3. And when I tried to redirect with https://targetweb.com?url=www.targetweb.com.attackersite.com
4. It will successfully redirected to the www.targetweb.com.attackersite.com website!
5. Due to the bad regex it has been successfully bypass their protection!

### Authors:
* [@bishal0x01](https://twitter.com/bishal0x01)

### Reference Tweets:
* https://twitter.com/bishal0x01/status/1262021038080053248
