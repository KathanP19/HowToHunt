# OTP Bypass (Registration & Authentication)

One-Time Passwords verify phone/email ownership during signup, login, and critical actions. Poor OTP implementations allow bypass of the verification step itself — meaning an attacker registers or authenticates without owning the phone/email. Impact: authentication bypass → account takeover of numbers the attacker doesn't control (P1–P2 when proven on arbitrary accounts).

> For OTP as a *second factor at login*, see `MFA_Bypasses/2FA_Bypass.md`. This file covers OTP verification in signup/login/action flows.

## Hunting Checklist

1. Map the OTP flow end to end: request/send endpoint, verify endpoint, resend endpoint, what identifiers bind them (phone, profileId, session).
2. On the verify step, inspect the response body for trust flags the client consumes (`verificationStatus`, `status`, `success`).
3. Test response manipulation with a deliberately wrong OTP.
4. Test the binding: can a valid OTP from Account A verify Account B? Swap phone/profile IDs between the send and verify requests.
5. Brute-force check: no rate limit / lockout on the verify endpoint (4–6 digit codes are cheap).
6. Try edge values: `null`, empty, `000000`, `123456`, `111111`, non-numeric.
7. Check the send/verify responses for OTP leakage in the body or JS.
8. Check expiry and one-time semantics: reuse an old/already-used OTP.
9. Test parameter confusion on the send request: change the destination number after request creation, arrays, HPP.
10. Forced browse the post-verification page/endpoint and test status-code tricks.

## Test Cases & Examples

### 1. Response Manipulation on OTP Verification (3 working methods)

**Method 1 — flip the verification flag:**
1. Register with a mobile number and request the OTP.
2. Enter an **incorrect OTP** and capture the server response in Burp.
3. Modify it before it reaches the browser:
```json
{"verificationStatus":false,"mobile":9072346577,"profileId":"84673832"}
```
```json
{"verificationStatus":true,"mobile":9072346577,"profileId":"84673832"}
```
4. Forward — the account authenticates despite the wrong OTP.

**Method 2 — replace the error with a success body:**
```json
{ "error": "Invalid OTP" }
```
```json
{ "success": "true" }
```

**Method 3 — cross-account status substitution:**
1. Register two accounts with different phone numbers.
2. Enter the **correct OTP** for Account 1; note the success response (`status:1`).
3. On Account 2 enter an **incorrect OTP** (`status:0`) and replace the response with the Account-1 success response.
4. OTP verification for Account 2 is bypassed.

### 2. Brute Force via No Rate Limiting
1. Request an OTP, enter a wrong code, capture the verify request:
```http
POST /api/otp/verify HTTP/1.1
Host: target.com
Content-Type: application/json

{"phone":"9072346577","otp":"000000"}
```
2. Send to Intruder → payload on `otp` → Numbers `000000–999999`.
3. No rate limit ⇒ the correct OTP is found (spot it via response-length/status difference).

### 3. Form Resubmission / Recipient Swap in Repeater
1. Start registration with a **non-existent phone number**; intercept the OTP-send request.
2. Send to Repeater, then change the phone number in the request to **your real number** and resend.
3. If the OTP for the fake number arrives on your phone, complete registration for the fake number → account tied to a number you don't control (or squatting victims' numbers).

### 4. Default / Static OTPs
```
000000, 111111, 123456
```
Test misconfigured defaults and check the client bundles:
```javascript
var otp = "123456";
```

### 5. OTP Leakage in Server Response
Intercept the response of the OTP-send request and grep the body for the just-sent code:
```json
{"message":"OTP sent","otp":"123456"}
```

### 6. Old / Reused OTPs
- After a successful verification, request a new OTP and try the **previous** one — if accepted, codes aren't invalidated on use.
- Verify with an OTP from an *expired* session to test the validity window.

## Bypass Techniques

- `null`/empty code and type juggling: `otp=` and `otp[]=1` (array) can slip past loose server comparisons.
- HPP on verify: `otp=000000&otp=123456` — validation and consumption may read different values.
- Skip the step: forced-browse the page/endpoint reached after verification; compare behavior when the verify call is never made.
- Status-code trust: a verify response of `4xx` rewritten to `200 OK` for clients that only check the status line (see `MFA_Bypasses/2FA_Bypass.md` #2).

## Tools & Resources

- Burp Suite (Repeater, Intruder with Numbers payload, Match-and-Replace), Cookie Editor for session binding tests.
- Related: `MFA_Bypasses/2FA_Bypass.md`, `Sign_Up_Functionality/Hunting_for_bugs_in_signup_feature.md`.

## Report Tips

- Prove bypass against an account whose phone/email you **do not control** — that converts "no rate limit" into demonstrated ATO.
- Show both the wrong-OTP attempt (server says no) and the manipulated/brute-forced client state (logged in) in one video.
- List the concrete defenses missing (rate limit, lockout, expiry, binding) so the fix is actionable for triage.

## Credits

- [@akshaykerkar13](https://twitter.com/akshaykerkar13), [@Yn0tWhy](https://twitter.com/Yn0tWhy), [@Virdoex_hunter](https://twitter.com/Virdoex_hunter), [@febinrev](https://twitter.com/febinrev), [@fani_malik](https://twitter.com/fanimalikhack), [@v3daxt](https://twitter.com/v3daxt), [@prakhar0x01](https://twitter.com/prakhar0x01); reformatted for HowToHunt by remonsec.
