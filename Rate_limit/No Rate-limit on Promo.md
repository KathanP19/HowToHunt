## No Rate-Limit on Promo

### Steps To Reproduce:
- 1) Go to URL - `https://abc.target.com/product/121/checkout/promo`
- 2) Navigate to `Offer/Promo/Coupon code` option
- 3) Enter the random digit
- 4) `Intercept the Request` and Send to intruder
- 5) Apply payload & `Start attack`

### Impact : 
- Financial Loss, an attacker can easily bruteforce all promo/coupon/Offer codes.