# Tabnabbing

Even though this vulnerability is rated by many bug bounty programs as a low severity bug, it is worth looking for it as it is very easy to find. 

### More information about the vulnerability:

When tabnabbing, the attacker searches for links that are inserted into the website and are under his control. Such links may be contained in a forum post, for example. Once he has found this kind of functionality, it checks that the link's `rel` attribute does not contain the value `noopener` and the target attribute contains the value `_blank`. If this is the case, the website is vulnerable to tabnabbing. 

### How to exploit: 
```
1. Attacker posts a link to a website under his control that contains the following JS code: window.opener.location = "http://evil.com"
2. He tricks the victim into visiting the link, which is opened in the browser in a new tab.
3. At the same time the JS code is executed and the background tab is redirected to the website evil.com, which is most likely a phishing website.
4. If the victim opens the background tab again and doesn't look at the address bar, it may happen that he thinks he is logged out, because a login page appears, for example.
5. The victim tries to log on again and the attacker receives the credentials
```

### How to search for it: 

As already mentioned, you have to search for the following link formats: 

```html
<a href="..." target="_blank" rel="" />  
or
<a href="..." target="_blank" />
```

### Author

* [@bolli95](https://github.com/bolli95)
