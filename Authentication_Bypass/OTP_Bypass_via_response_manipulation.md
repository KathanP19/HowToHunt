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
