
#### What is CSP?

CSP stands for `Content Security Policy` which is a mechanism to define which resources can be fetched out or executed by a web page. In other words, it can be understood as a policy that decides which scripts, images, iframes can be called or executed on a particular page from different locations. Content Security Policy is implemented via response headers or meta elements of the HTML page. From there, it’s browser’s call to follow that policy and actively block violations as they are detected.

#### How does it work?

CSP works by restricting the origins that active and passive content can be loaded from. It can additionally restrict certain aspects of active content such as the execution of inline JavaScript, and the use of eval().

#### Defining Resources

```
default-src 'none';
img-src 'self';
script-src 'self' https://code.jquery.com;
style-src 'self';
report-uri /__cspreport__
font-src 'self' https://addons.cdn.mozilla.net;
frame-src 'self' https://ic.paypal.com https://paypal.com;
media-src https://videos.cdn.mozilla.net;
object-src 'none';
```
#### Some defining resources

_Some definitions_

1. **script-src:** This directive specifies allowed sources for JavaScript. This includes not only URLs loaded directly into  elements, but also things like inline script event handlers (onclick) and XSLT stylesheets which can trigger script execution. 
2. **default-src:** This directive defines the policy for fetching resources by default. When fetch directives are absent in CSP header the browser follows this directive by default.
3. **Child-src:** This directive defines allowed resources for web workers and embedded frame contents. 
4. **connect-src:** This directive restricts URLs to load using interfaces like ,fetch,websocket,XMLHttpRequest 
5. **frame-src:** This directive restricts URLs to which frames can be called out. 
6. **frame-ancestors:** This directive specifies the sources that can embed the current page. This directive applies to , , , and  tags. This directive can't be used in  tags and applies only to non-HTML resources. 
7. **img-src:** It defines allowed sources to load images on the web page. 
8. **Manifest-src:** This directive defines allowed sources of application manifest files. 
9. **media-src:** It defines allowed sources from where media objects like , and  can be loaded. 
10. **object-src:** It defines allowed sources for the , and  elements.
11. **base-uri:** It defines allowed URLs which can be loaded using  element. 
12. **form-action:** This directive lists valid endpoints for submission from  tags.
13. **plugin-types:** It defineslimits the kinds of mime types a page may invoke. 
upgrade-insecure-requests: This directive instructs browsers to rewrite URL schemes, changing HTTP to HTTPS. This directive can be useful for websites with large numbers of old URL's that need to be rewritten.
14. **sandbox:** sandbox directive enables a sandbox for the requested resource similar to the  sandbox attribute. It applies restrictions to a page's actions including preventing popups, preventing the execution of plugins and scripts, and enforcing a same-origin policy.



### Basic CSP Bypass

There are quit a few ways to mess up your implementation of CSP. One of the easiest ways to misconfigure CSP is to use dangerous values when setting policies. For example suppose you have the following CSP header:

```default-src 'self' *```

As you know the default-src policy acts a catch all policy. You also know that * acts as a wild card. So this policy is basically saying allow any resources to be loaded. Its the same thing as not having a CSP header! You should always look out for wildcard permissions.

Lets look at another CSP header:
```
script-src 'unsafe-inline' 'unsafe-eval' 'self' data: https://www.google.com http://www.google-analytics.com/gtm/js  https://*.gstatic.com/feedback/ https://accounts.google.com;
```

Here we have the policy script-src which we know is used to define where we can load javascript files from. Normally things like ***<IMG SRC=”javascript:alert(‘XSS’);”>*** would be blocked but due to the value ‘unsafe-inline’ this will execute. This is something you always want to look out for as it is very handy as an attacker.

You can also see the value data: this will allow you to load javascript if you have the data: element as shown below: <iframe/src=”data:text/html,<svg onload=alert(1)>”>.

So far all of the techniques used to bypass CSP have been due to some misconfiguration or abusing legitimate features of CSP. There are also a few other techniques which can be used to bypass the CSP.

### JSONP CSP Bypass

If you don’t know what JSONP is you might want to go look at a few tutorials on that topic but ill give you a brief overview. JSONP is a way to bypass the same object policy (SOP). A JSONP endpoint lets you insert a javascript payload , normally in a GET parameter called “callback” and the endpoint will then return your payload back to you with the content type of JSON allowing it to bypass the SOP. Basically we can use the JSONP endpoint to serve up our javascript payload. You can find an example below:
```
https://accounts.google.com/o/oauth2/revoke?callback=alert(1337)
```

As you can see above we have our alert function being displayed on the page.

The danger comes in when a CSP header has one of these endpoints whitelisted in the script-src policy. This would mean we could load our malicious javascript via the JSONP endpoint bypassing the CSP policy.

Look at the following CSP header:
```
script-src https://www.google.com http://www.google-analytics.com/gtm/js  https://*.gstatic.com/feedback/ https://accounts.google.com;
```
This would get blocked by the CSP

```something.example.com?vuln_param=javascript:alert(1);```

This would pass because accounts.google.com is allowed to load javascript files. However, we are abusing the JSONP feature to load our malicious javascript.

```something.example.com?vuln_param=https://accounts.google.com/o/oauth2/revoke?callback=alert(1337)```

#### CSP Injection Bypass

The third type of CSP bypass is called CSP injection. This occurs when user supplied input is reflected in the CSP header. Suppose you have the following url:

```example.com?vuln=something_vuln_csp```

-> If your input is reflected in the CSP header you should have somthing like this.

```
script-src something_vuln_csp;
object-src 'none';
base-uri 'none';
require-trusted-types-for 'script';
report-uri https://csp.example.com;
```

This means we can control what value the script-src value is set to. We can easily bypass the CSP by setting this value to a domain we control.

## Author:
* [@harsha0x01](https://twitter.com/harsha0x01)
