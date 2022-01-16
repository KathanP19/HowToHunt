# Rate Limit Bypass Techniques 
## There are two ways to do that 
- Customizing HTTP Methods
- Adding Headers to Spoof IP

## 1. Customizing HTTP Methods
- If the request goes on GET try to change it to POST, PUT, etc.,
- If you wanna bypass the rate-limit in API's try HEAD method.

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
## Adding HTTP Headers to Spoof IP and Evade Detection
- These are Headers I've collected so far to Bypass Rate-Limits.
```
X-Forwarded: 127.0.0.1
X-Forwarded-By: 127.0.0.1
X-Forwarded-For: 127.0.0.1
X-Forwarded-For-Original: 127.0.0.1
X-Forwarder-For: 127.0.0.1
X-Forward-For: 127.0.0.1
Forwarded-For: 127.0.0.1
Forwarded-For-Ip: 127.0.0.1
X-Custom-IP-Authorization: 127.0.0.1
X-Originating-IP: 127.0.0.1
X-Remote-IP: 127.0.0.1
X-Remote-Addr: 127.0.0.1
```

## Rate Limit Bypass using Special Characters 

- Adding Null Byte ( %00 ) at the end of the Email can sometimes Bypass Rate Limit.
- Try adding a Space Character after a Email. ( Not Encoded )
- Some Common Characters that help bypassing Rate Limit : %0d , %2e , %09 , %20 , %0, %00, %0d%0a, %0a, %0C
- Adding a slash(/) at the end of api endpoint can also Bypass Rate Limit. `domain.com/v1/login` -> `domain.com/v1/login/`


## Using IP Rotate Burp Extension

- Try changing the user-agent, the cookies... anything that could be able to identify you
- If they are limiting to 10 tries per IP, every 10 tries change the IP inside the header.
  Change other headers
- Burp Suite's Extension IP Rotate works well in many cases. Make sure you have Jython installed along.

- Here You'll everything you need - https://github.com/PortSwigger/ip-rotate


## You can find some more here - [Check this out](https://medium.com/bugbountywriteup/bypassing-rate-limit-like-a-pro-5f3e40250d3c)
## You can find more with screenshot https://medium.com/@huzaifa_tahir/methods-to-bypass-rate-limit-5185e6c67ecd

# Reference
* https://twitter.com/m4ll0k2/status/1294983599943540738/photo/1
* https://twitter.com/SalahHasoneh1/status/1287366496432332800
* https://twitter.com/SMHTahsin33/status/1295054667613757441 (all in one must check)

# Authors:  
* [Keshav Malik](https://www.linkedin.com/in/keshav-malik-22478014a) </br>
* [0xd3vil](https://linkedin.com/in/0xd3vil) </br>
* [Virdoex_hunter](https://twitter.com/Virdoex_hunter)
* [@0xCyberPirate](https://twitter.com/0xCyberPirate)
