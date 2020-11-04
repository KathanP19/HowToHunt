# OTP Bypass on Login via Response manipulation

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

twitter @Yn0tWhy
