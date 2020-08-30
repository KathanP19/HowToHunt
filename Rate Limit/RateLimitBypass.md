# Rate Limit Bypass Techniques 

## Rate Limit Bypass using Header 

Use the following Header just Below the Host Header 

```
X-Forwarded-For: IP
X-Forwarded-IP: IP
X-Client-IP: IP
X-Remote-IP: IP
X-Originating-IP: IP
X-Host: IP
X-Client: IP

```

## Rate Limit Bypass using Sepcial Characters 

- Adding Null Byte ( %00 ) at the end of the Email can sometimes Bypass Rate Limit.
- Try adding a Space Character after a Email. ( Not Encoded )
- Some Common Characters that help bypassing Rate Limit : %0d , %2e , %09 , %20 , %0


## Using IP Rotate Burp Extension

- Burp Suite's Extension IP Rotate works well in many cases. Make sure you have Jython installed along.

- Here You'll everything you need - https://github.com/PortSwigger/ip-rotate


## You can find some more here - [Check this out](https://medium.com/bugbountywriteup/bypassing-rate-limit-like-a-pro-5f3e40250d3c)

Author: [Keshav Malik](https://www.linkedin.com/in/keshav-malik-22478014a)
