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
 
# Stored Xss Methods
Stored Xss are mostly found manually
 1. Enumerate the Firewall using above Methods and select a payload to test accordingly.
 2. Try that selected WAF bypass payload while registering on a site in fields like username, name, address, email, etc.
 3. Try Payload in File name of profile picture and also in the source file of image.
 4. Try in Comment section anywhere on target site.
 5. Try on every input fields which get reflected in page and which can be seen by other users.

# Blind Xss
Similar to Reflected Xss Or Stored Xss But you Dont get any reflection, but you get response on you server.

 1. Similar methods As given above except try putting payload which can give a callback on your server when executed.
 2. You can Used [https://xsshunter.com/](https://xsshunter.com/) or Use burpcollaborator or ngrok.
 3. Try it on contact forms or similar functionality.

### Authors
* [@KathanP19](https://twitter.com/KathanP19)
