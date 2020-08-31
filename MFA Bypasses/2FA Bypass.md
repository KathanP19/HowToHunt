# 2FA Bypass

* Reset Password function	
* Rate limit
* Sendin all alphabets instead of number
* 2FA bypass by substituting part of the request from the session of another account. 
    ```
    If a parameter with a specific value is sent to verify the code in the request, try sending the value from the request of another account.
    
    For example, when sending an OTP code, the form ID/user ID or cookie is checked, which is associated with sending the code. If we apply the data from the parameters of the account on which you want to bypass code verification (Account 1) to a session of a completely different account (Account 2), receive the code and enter it on the second account, then we can bypass the protection on the first account. After reloading the page, 2FA should disappear.
    ```
 * Bypass 2FA using the “memorization” functionality.
		
    `Many sites that support 2FA, have a “remember me” functionality. It is useful when the user doesn’t want to enter a 2FA code on subsequent login windows. And it is important to identify the way in which 2FA is “remembered”. This can be a cookie, a value in session/local storage, or simply attaching 2FA to an IP address.`
 * Information Disclousre(otp leak in response)
 * Bypassing 2fa Via OAuth mechanism ( Mostly not Applicable one )
		
    `Site.com requests Facebook for OAuth token > Facebook verifies user account > Facebook send callback code > Site.com logs a user in (Rare case)`
 * Bypassing 2fa using response manipulation
   ```
   Enter correct OTP -> Intercept & capture the response -> logout -> enter wrong OTP -> Intercept & change the response with successful previous response -> logged in
   ```
 * Bypassing 2fa via CSRF attack on disable 2FA
    ```
    Signup for two account -> Login into attacker account & capture the disable 2FA request -> generate CSRF POC with .HTML extension -> Login into victim account and fire the request — — -> It disable 2FA which leads to 2FA Bypass.
    ```
