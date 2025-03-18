# **2FA Bypass Techniques**

## **Introduction**
Two-Factor Authentication (2FA) is a security mechanism designed to add an extra layer of protection by requiring users to provide an additional verification code after entering their credentials. However, improper implementations of 2FA can introduce various security flaws that allow attackers to bypass authentication.

This document outlines **common 2FA bypass techniques**, including **response manipulation, brute-force attacks, backup code abuse, and session hijacking**. Each method is detailed with examples and exploitation steps.

For a **visual reference**, a **[2FA Bypass Mindmap](https://mm.tt/1736437018?t=SEeZOmvt01)** provides an overview of different attack vectors.

---

## **Common 2FA Bypass Techniques**

### **Index of Techniques**
| #  | **Technique** |
|----|--------------|
| **1**  | Response Manipulation |
| **2**  | Status Code Manipulation |
| **3**  | 2FA Code Leakage in Response |
| **4**  | JavaScript File Analysis |
| **5**  | 2FA Code Reusability |
| **6**  | Lack of Brute-Force Protection |
| **7**  | Missing 2FA Code Integrity Validation |
| **8**  | CSRF on 2FA Disabling |
| **9**  | Password Reset Disables 2FA |
| **10** | Backup Code Abuse |
| **11** | Clickjacking on 2FA Disabling Page |
| **12** | Enabling 2FA Without Expiring Active Sessions |
| **13** | Bypass 2FA with `null` or `000000` |

---

## **1. Response Manipulation**
Some 2FA implementations return a JSON response indicating whether authentication was successful. **Altering the response** can bypass restrictions.

### **Exploitation**
- Intercept the response using **Burp Suite** or **a browser's developer tools**.
- Look for a response like:
  ```json
  { "success": false }
  ```
- Change it to:
  ```json
  { "success": true }
  ```
- If client-side validation is weak, access is granted.

---

## **2. Status Code Manipulation**
Some applications rely on HTTP status codes to determine authentication success.

### **Exploitation**
- If a **4xx error** (e.g., `401 Unauthorized`) is received after entering a **wrong** 2FA code, modify the response to:
  ```
  HTTP/1.1 200 OK
  ```
- Some applications may grant access **even if authentication failed**.

---

## **3. 2FA Code Leakage in API Responses**
Some applications accidentally **leak the 2FA code** in their API response.

### **Exploitation**
- Intercept the **request triggering the 2FA code**.
- Examine the API response.
- If the response contains:
  ```json
  { "otp": "123456" }
  ```
  - The attacker can directly **use the leaked OTP**.

---

## **4. JavaScript File Analysis**
Some applications store **2FA-related logic** in JavaScript files.

### **Exploitation**
- Check for exposed `.js` files in the application.
- Look for sensitive **hardcoded values** like:
  ```javascript
  var otp = "123456";
  ```
- Attackers can **extract OTP verification logic** or **static OTPs**.

---

## **5. 2FA Code Reusability**
Some applications **do not expire OTPs after use**, allowing attackers to **reuse** them.

### **Exploitation**
- Obtain a **valid OTP** from a previous session.
- Attempt to reuse the same OTP for authentication.
- If the system does not enforce **one-time use**, the **old OTP grants access**.

---

## **6. Lack of Brute-Force Protection**
Applications that **do not limit OTP attempts** allow brute-forcing.

### **Exploitation**
- Identify the **number of OTP digits** (commonly `4`-`6`).
- Use a tool like `Burp Intruder` to brute-force:
  ```
  000000 - 999999
  ```
- **Weak OTP validation** allows attackers to guess the correct OTP.

---

## **7. Missing 2FA Code Integrity Validation**
Some systems accept **any valid OTP**, even from different accounts.

### **Exploitation**
- Obtain a **valid OTP** for **Account A**.
- Use the **same OTP** to authenticate **Account B**.
- If the system **does not verify OTP ownership**, access is granted.

---

## **8. CSRF on 2FA Disabling**
Some applications **lack CSRF protection** when disabling 2FA.

### **Exploitation**
- Construct a **malicious request** to disable 2FA:
  ```html
  <form action="https://victim-site.com/disable-2fa" method="POST">
      <input type="hidden" name="disable" value="true">
      <input type="submit" value="Click to win a prize!">
  </form>
  ```
- Trick the victim into **clicking the form**, disabling their 2FA.

---

## **9. Password Reset Disables 2FA**
Some systems **disable 2FA** when a user resets their password.

### **Exploitation**
- If an account has 2FA enabled, attempt a **password reset**.
- Check if **2FA is still active** after resetting the password.
- If **2FA is disabled**, log in **without 2FA authentication**.

---

## **10. Backup Code Abuse**
Backup codes provide **alternative login options** when OTP is unavailable.

### **Exploitation**
- If backup codes are stored **insecurely**, they can be leaked or stolen.
- Some applications **do not expire backup codes after use**, allowing repeated exploitation.

---

## **11. Clickjacking on 2FA Disabling Page**
Some applications allow **2FA to be disabled** without additional verification.

### **Exploitation**
- Load the **2FA disabling page** in an `<iframe>`.
- Trick the victim into **clicking the iframe** (e.g., by overlaying it over an attractive button).
- **2FA is disabled without the victim realizing it.**

---

## **12. Enabling 2FA Does Not Expire Active Sessions**
In some applications, **enabling 2FA** does not log out active sessions.

### **Exploitation**
- If an attacker hijacks a **session before 2FA is enabled**, they **retain access** even after 2FA is enforced.
- Attackers can **maintain persistence** despite 2FA protection.

---

## **13. Bypassing 2FA with `null` or `000000`**
Some poorly implemented 2FA mechanisms accept **default or empty codes**.

### **Exploitation**
- Enter `null`, `000000`, or similar **default values** in the OTP field.
- If the system **accepts these values**, authentication is bypassed.

---

## **Further Reading**
- **[Testing Two-Factor Authentication](https://research.nccgroup.com/2021/06/10/testing-two-factor-authentication/)** by **NCC Group**

---

## **Authors**
- **[Harsh Bothra](https://twitter.com/harshbothra_)**
- **[Vishal Saini](https://twitter.com/k4k4r07)**

---
*Enhanced and reformatted for HowToHunt repository by [remonsec](https://x.com/remonsec)*
