# OTP Bypass on Register account via Response manipulation

## Steps:-
- Register account with mobile number and request for OTP.
- Enter incorrect OTP and capture the request in Burpsuite.
- Do intercept response to this request and forward the request.
- response will be 

`{"verificationStatus":false,"mobile":9072346577","profileId":"84673832"}`

- Change this response to

`{"verificationStatus":true,"mobile":9072346577","profileId":"84673832"}`

- And forward the response.
- You will be logged in to the account.


`Impact:` Account Takeover

---

## Steps:-
- Go to login and wait for OTP pop up.
- Enter incorrect OTP and capture the request in Burpsuite.
- Do intercept response to this request and forward the request.
- response will be 

`error`

- Change this response to

`success`

- And forward the response.
- You will be logged in to the account.


`Impact:` Account Takeover

---
## Steps:
 
  ```
    1.Register 2 accounts with any 2 mobile number(first enter right otp)
    2.Intercept your request
    3.click on action -> Do intercept -> intercept response to this request.
    4.check what the message will display like status:1
    5.Follow the same procedure with other account but this time enter wrong otp
    6.Intercept respone to the request
    7.See the message like you get status:0
    8.Change status to 1 i.e, status:1 and forward the request if you logged in means you just done authentication bypass.
  ```
  
### Author:
* [@akshaykerkar13](https://twitter.com/akshaykerkar13)
* [@Yn0tWhy](https://twitter.com/Yn0tWhy)
* [@Virdoex_hunter](https://twitter.com/Virdoex_hunter)
