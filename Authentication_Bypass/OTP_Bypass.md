# **OTP Bypass Techniques in Account Registration and Authentication**

## **Introduction**
One-Time Passwords (OTP) are commonly used for authentication and verification in account registration, login, and critical actions. However, poor OTP implementations can lead to **authentication bypass, account takeover, and unauthorized access**.

This document outlines **various OTP bypass techniques**, including **response manipulation, rate limit exploitation, default OTP usage, and session validation flaws**.

---

## **OTP Bypass via Response Manipulation**
### **Method 1: Manipulating OTP Verification Response**
#### **Steps:**
1. Register an account with a mobile number and request an OTP.
2. Enter an **incorrect OTP** and capture the request using **Burp Suite**.
3. Intercept and **modify the server's response**:
   - Original response:
     ```json
     {"verificationStatus":false,"mobile":9072346577,"profileId":"84673832"}
     ```
   - Change to:
     ```json
     {"verificationStatus":true,"mobile":9072346577,"profileId":"84673832"}
     ```
4. Forward the manipulated response.
5. The system authenticates the account despite the incorrect OTP.

**Impact:**  
- **Full account takeover** without providing a valid OTP.

---

### **Method 2: Changing Error Response to Success**
#### **Steps:**
1. Go to the **login page** and enter your phone number.
2. When prompted for an OTP, enter an **incorrect OTP**.
3. Capture the **server response**:
   ```json
   { "error": "Invalid OTP" }
   ```
4. Modify it to:
   ```json
   { "success": "true" }
   ```
5. Forward the response.
6. If the server accepts this modification, you gain access without entering a valid OTP.

**Impact:**  
- **Authentication bypass leading to account takeover**.

---

### **Method 3: OTP Verification Across Multiple Accounts**
#### **Steps:**
1. Register **two different accounts** with separate phone numbers.
2. **Enter the correct OTP** for one account and intercept the request.
3. Capture the server response and note **status:1** (success).
4. Now, attempt to verify the second account with an **incorrect OTP**.
5. Intercept the server response where the status is **status:0** (failure).
6. Change **status:0** to **status:1** and forward the response.
7. If successful, you bypass OTP authentication.

**Impact:**  
- **Bypassing OTP verification for multiple accounts**.

---

## **OTP Bypass Using Form Resubmission in Repeater**
#### **Steps:**
1. Register an account using a **non-existent phone number**.
2. Intercept the OTP request in **Burp Suite**.
3. Send the request to **Repeater** and forward it.
4. Modify the phone number in the request to **your real number**.
5. If the system **sends the OTP to your real number**, use it to register under the **fake number**.

**Impact:**  
- **Unauthorized account registration using someone else's OTP**.

---

## **Bypassing OTP with No Rate Limiting**
### **Steps:**
1. **Create an account** and request an OTP.
2. Enter an **incorrect OTP** and capture the request in Burp Suite.
3. Send the request to **Burp Intruder** and **set a payload on the OTP field**.
4. Set **payload type as numbers** (`000000` to `999999`).
5. Start the attack.
6. If **no rate limit** is enforced, the correct OTP will eventually match.

**Impact:**  
- **Complete OTP bypass through brute force**.

---

## **Additional OTP Bypass Test Cases**
### **1. Default OTP Values**
- Some applications use default OTP values such as:
  ```
  111111, 123456, 000000
  ```
- Test common default values to check for misconfigurations.

### **2. OTP Leakage in Server Response**
- Some applications leak OTPs in API responses.
- **Intercept OTP request responses** and check if OTP is present.

### **3. Checking if Old OTP is Still Valid**
- Some systems allow the **reuse of old OTPs**.
- Test if **previously used OTPs** are still accepted.

---

## **Rate Limiting Attack on OTP Verification**
### **Steps:**
1. **Navigate to the OTP verification endpoint**:
   ```
   https://abc.target.com/verify/phoneno
   ```
2. Enter an **invalid OTP** (e.g., `000000`).
3. **Intercept the request** and send it to **Intruder**.
4. Set the **OTP field as the payload position**.
5. Use **payload type: numbers** and define a **range (000000 - 999999)**.
6. Start the attack.
7. Identify a **response length change**, which may indicate the correct OTP.

**Impact:**  
- **Brute-force attack leading to OTP bypass and account takeover**.

---

## **Contributors**
- **[@akshaykerkar13](https://twitter.com/akshaykerkar13)**
- **[@Yn0tWhy](https://twitter.com/Yn0tWhy)**
- **[@Virdoex_hunter](https://twitter.com/Virdoex_hunter)**
- **[@febinrev](https://twitter.com/febinrev)**
- **[@fani_malik](https://twitter.com/fanimalikhack)**
- **[@v3daxt](https://twitter.com/v3daxt)**
- **[@prakhar0x01](https://twitter.com/prakhar0x01)**

---
*Enhanced and reformatted for HowToHunt repository by [remonsec](https://x.com/remonsec)*
