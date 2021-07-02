# Reflected Xss Methods
Many methods out in wild but here are few most common , but not limited.

One Awesome mind map for approach to reflected xss can be found here [https://github.com/A9HORA/Reflected-XSS-Mindmap](https://github.com/A9HORA/Reflected-XSS-Mindmap) Made By [@A9HORA](https://twitter.com/A9HORA)

***Tip: While using other methods put method 2 in background in terminal or on vps***

### 1. Using Burp
 1. Download Reflection and sentinal plugin for burp.
 2. Walk and spider the target site.
 3. Check the reflected params tab in burp
 4. send that sentinal or check manually.
 
### 2. Using Waybackurls and other similar site
 1. Use [Gau](https://github.com/lc/gau) or [Wayback urls](https://github.com/tomnomnom/waybackurls) to passively gather urls of the target.
 2. Filter the parameters using `grep "="` or gf patterns and store it in a new file.
 3. Now run [Gxss](https://github.com/KathanP19/Gxss) or [bxss](https://github.com/ethicalhackingplayground/bxss/) on that new file.
 4. Check Reflected Param Manually or use some tool like [dalfox](https://github.com/hahwul/dalfox) 

### 3. Using Google Dorks
 1. Using Google Dork `site:target.com` filter the result
 2. Now search for links which have params by adding more dorks something like `site:target.com inurl:".php?"` or `site:target.com filetype:php` etc
    you can find some dorks at this link [https://www.openbugbounty.org/blog/devl00p/top-100-xss-dorks/](https://www.openbugbounty.org/blog/devl00p/top-100-xss-dorks/) or google it out.
 3. Check if the param value is getting reflected in html source code 
 4. Try Xss payload there or pass it to some tool
 
### 4. Find Hidden Variables In Source Code.
 1. Check Javascript file or html Source file for hidden or unused variables 
 2. You can Manually Check Right Click View Page Source and search for `var=` , `=""` , `=''`.
 3. Now Append that to webpage urls. For example `https://example.com?hiddenvariablename=xss`.
 
### 5. Other Methods
 1. Use Methods 1 or 2 to Gather the urls
 2. Enumerate the Firewall using [https://github.com/Ekultek/WhatWaf](https://github.com/Ekultek/WhatWaf) or other similar tool.
 3. Find WAF bypass payload on twitter by searching or in this Github Repo [https://github.com/0xInfection/Awesome-WAF](https://github.com/0xInfection/Awesome-WAF)
 4. Also Use [Arjun](https://github.com/s0md3v/Arjun) to find hidden params.

*Tips*
- Check the error pages (404,403,..) sometimes they contain reflected values
	- Trigger a 403 by trying to get the .htaccess file
- Try every reflected parameter

*Video's*
- https://www.youtube.com/watch?v=wuyAY3vvd9s
- https://www.youtube.com/watch?v=GsyOuQBG2yM
- https://www.youtube.com/watch?v=5L_14F-uNGk
- https://www.youtube.com/watch?v=N3HfF6_3k94
 
# Stored Xss Methods
Stored Xss are mostly found manually
 1. Enumerate the Firewall using above Methods and select a payload to test accordingly.
 2. Try that selected WAF bypass payload while registering on a site in fields like username, name, address, email, etc.
 3. Try Payload in File name of profile picture and also in the source file of image.
 4. Try in Comment section anywhere on target site.
 5. Try on every input fields which get reflected in page and which can be seen by other users.
 6. Try to signup using your name + xss payload and that can lead to stored xss.
*Tips*
- For every input field
	- Try to get ```<a href=#>test</a>``` an entity in
	- Try to get an obfuscated entity in
	- If it catches on anything, go deeper

*Video's*
- https://www.youtube.com/watch?v=uHy1x1NkwRU
Writeup:
-https://medium.com/@fatin151485/how-i-found-my-first-stored-xss-on-popular-eboighar-com-6bd497b0bb96

# Blind Xss
Similar to Reflected Xss Or Stored Xss But you Dont get any reflection, but you get response on you server.

 1. Similar methods As given above except try putting payload which can give a callback on your server when executed.
 2. You can Used [https://xsshunter.com/](https://xsshunter.com/) or Use burpcollaborator or ngrok.
 3. Try it on contact forms or similar functionality.

*Tips*
- Copy every payload from your xsshunter payloads section and paste it into every field you see
- XSS hunter contains a payload for CSP bypass
- Generate some variations of your payloads (example replace < with `&lt;`)

### Where to look for Blind XSS……
```
1- Review forms
2- Contact Us pages
3- Passwords(You never know if the other side doesn’t properly handle input and if your password is in View mode)
4- Address fields of e-commerce sites
5- First or Last Name field while doing Credit Card Payments
6- Set User-Agent to a Blind XSS payload. You can do that easily from a proxy such as Burpsuite.
7- Log Viewers
8- Feedback Page
9- Chat Applications
10- Any app that requires user moderation
```

# DOM XSS

*Tips*
- Would not recommend manually looking for DOM XSS
- Burp suite PRO scanner can find DOM XSS
- Tool: https://github.com/dpnishant/ra2-dom-xss-scanner

*Video's*
- https://www.youtube.com/watch?v=gBqzzhgHoYg
- https://www.youtube.com/watch?v=WclmtS8Ftc4

# XSS filter evasion tips

*Tips*
- < and > can be replace with html entities `&lt;` and `&gt;`
- You can try an XSS polyglot
	- ```javascript:/*--></title></style></textarea></script></xmp><svg/onload='+/"/+/onmouseover=1/+/[*/[]/+alert(1)//'>```
	- https://gist.github.com/michenriksen/d729cd67736d750b3551876bbedbe626
	
### XSS Firewall Bypass Techniques

* Check if the firewall is blocking only lowercase
```
Ex:- <scRipT>alert(1)</scRipT>
```
* Try to break firewall regex with the  new line(\r\n)
```
Ex:- <script>%0alert(1)</script>
```
* Try Double Encoding
```
Ex:- %2522
```
* Testing for recursive filters, if firewall removes text in red, we will have clear payload
```
Ex:- <src<script>ipt>alert(1);</scr</script>ipt>
```
* Injecting anchor tag without whitespaces
```
Ex:- <a/href="j&Tab;a&Tab;v&Tab;asc&Tab;ri&Tab;pt:alert&lpar;1&rpar;">
```
* Try to bypass whitespaces using Bullet
```
Ex:- <svg•onload=alert(1)>
```
* Try to change request method
```
Ex:- GET /?q=xss  POST/
                  q=xss
```
* Try CRLF Inection
```
Ex:- GET /%0A%ODValue=%20Virus
     POST 
     Value= Virus
 ```
## Thanks To
* [The XSS rat](https://www.youtube.com/channel/UCjBhClJ59W4hfUly51i11hg)
* [sratarun](https://twitter.com/sratarun)

## Reference
* [https://github.com/heilla/SecurityTesting/blob/master/HuntingCheckList.md](https://github.com/heilla/SecurityTesting/blob/master/HuntingCheckList.md)

### Authors
* [@KathanP19](https://twitter.com/KathanP19)
* [@harsha0x01](https://twitter.com/harsha0x01)
