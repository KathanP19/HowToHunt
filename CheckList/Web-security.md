## Basic web security checklist
```
1. Ensure HSTS is enabled.
2. Ensure X-frame-options is set/CSP frame-anchester is set(better way to do).
3. Ensure Sameorgin policy is Set.
4. Ensure CORS is set up only for trusted domains.
5. Ensure CSP is Implemented in a secure way Guide to setup secure CSP
6. Ensure X-XSS-Protection header is set (not as 0).
7. Ensure Cookie security flags are set,
   Secure Flag and Httponly set as true
   Same site set as Lax or strict depends on nature of the Webapp (but not as set as none)
8. Ensure ETag token is set if the website requires to keep updating the resources.
9. Ensure feature policy is set.
10. Ensure X-Content-Type-Options is set.
11. Disable XML features that the application does not intend to use(prevents XXE)
12. Use the security headers page to check what headers are missing!
```
