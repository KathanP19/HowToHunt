## Bypassing OTP in registration forms by repeating the form submission multiple times using repeater

#### Steps :
-----------------------------------------------------------------------------------------------------------------------------------------------
      1. Create an account with a non-existing phone number
      2. Intercept the Request in BurpSuite
      3. Send the request to the repeater and forward
      4. Go to Repeater tab and change the non-existent phone number to your phone number
      5. If you got an OTP to your phone, try using that OTP to register that non-existent number
      
------------------------------------------------------------------------------------------------------------------------------

## Author
[febinrev](https://twitter.com/febinrev)
