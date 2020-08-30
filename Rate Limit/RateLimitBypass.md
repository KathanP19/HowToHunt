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

#or use double X-Forwarded-For header
X-Forwarded-For:
X-Forwarded-For: IP
```

## Rate Limit Bypass using Special Characters 

- Adding Null Byte ( %00 ) at the end of the Email can sometimes Bypass Rate Limit.
- Try adding a Space Character after a Email. ( Not Encoded )
- Some Common Characters that help bypassing Rate Limit : %0d , %2e , %09 , %20 , %0, %00, %0d%0a, %0a, %09, %0C


## Using IP Rotate Burp Extension

- Try changing the user-agent, the cookies... anything that could be able to identify you
- If they are limiting to 10 tries per IP, every 10 tries change the IP inside the header.
  Change other headers
- Burp Suite's Extension IP Rotate works well in many cases. Make sure you have Jython installed along.

- Here You'll everything you need - https://github.com/PortSwigger/ip-rotate


## You can find some more here - [Check this out](https://medium.com/bugbountywriteup/bypassing-rate-limit-like-a-pro-5f3e40250d3c)

Author: [Keshav Malik](https://www.linkedin.com/in/keshav-malik-22478014a) </br>
Author: [0xd3vil](https://linkedin.com/in/0xd3vil)
