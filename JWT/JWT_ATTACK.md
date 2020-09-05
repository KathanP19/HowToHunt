# JWT Attack

### FIRST IF YOU DON'T KNOW WHAT IS JWT YOU MUST READ AND WATCH BELOW RESOURCES
-----------------------------------------------------------------------
* https://twitter.com/BHinfoSecurity/status/1299743624553549825?s=09
* https://youtu.be/ghfmx4pr1Qg ( very begginer friendly)
* https://medium.com/ag-grid/a-plain-english-introduction-to-json-web-tokens-jwt-what-it-is-and-what-it-isnt-8076ca679843
* https://medium.com/swlh/hacking-json-web-tokens-jwts-9122efe91e4a
 
### NOTES FOR ATTACKING JWT
* What the heck is this ?!
```
1. It is an authentication type 
2. It consists of header,payload,Signature
```
---------------------------------------------------------------------------------
* Header 	
```
{
 "alg" : "HS256",
 "typ" : "JWT"
}
```
-------------------------------------------------------------------
* Payload 	
```
{
 "loggedInAs" : "admin",
 "iat" : 1422779638
}
```
-----------------------------------------------------------------------------
* Signature 	
```
HMAC-SHA256
(
 secret,
 base64urlEncoding(header) + '.' +
 base64urlEncoding(payload)
)
```
-----------------------------------------------
* Changing alg to null 
* Example
```
{
 "alg" : "NONE",
 "typ" : "JWT"
}
Note;;////--remove the signuature
You can also use none,nOne,None,n0Ne
```
-------------
* Change the payload like 
```
Payload 	

{
 "loggedInAs" : "admin", 
 "iat" : 1422779638
}
```
* Here change user to admin
----------------------------------------------------
 # SOME MORE TIPS AND METHOD
 --------------------------------------------------------
 1. First decode full token or 1 1 each part of token to base64
 2. Change the payload use jwt web token burp
 3. Changing encrption  rs256 to sh256
 4. Signature not changes remove it or temper it,
 5. Brute forcing the key in hs256 because it use same key to sign and verify means publickey=private key
 ---------------------------------------------------------------------------------------------------
 # TOOLS TO USE
 -----------------------------------------------------------------------------------------------
 * Jwt token attack burp extention
 
      (Link - [https://github.com/portswigger/json-web-token-attacker](https://github.com/portswigger/json-web-token-attacker))
 * Base64 decoder
 * jwt.io to analyse the struct of token
 * jwt cat for weak secret token
      
      (Link: [https://github.com/aress31/jwtcat](https://github.com/aress31/jwtcat))
      
      (Link : [https://github.com/ticarpi/jwt_tool.git](https://github.com/ticarpi/jwt_tool.git))

---------------------------------------------------------------------------------------------------------------------------
### SOURCES: 
* Youtube,Medium,Github,Google
### Author
* [Naman Shah](https://twitter.com/naman_1910)
 
