# Hidden API Functionality Exposure
- Application programming interfaces (APIs) have become a critical part of almost every business. APIs are responsible for transferring information between systems within a company or to external companies. For example, when you log in to a website like Google or Facebook, an API processes your login credentials to verify they are correct.

1. Swagger UI Documentation
2. Dictionary Attack | Brute force
3. Common wordlist for API Enum :
- https://wordlists.assetnote.io/
- https://github.com/Net-hunter121/API-Wordlist

## Steps to Perform This Attack :
```
Step 1 : Capture the request into Burp, Send the request to repeater and intruder tab
Step 2 : Add the endpoint into the intruder tab and add the payload from the word-list
Step 3 : 1st use dictionary attack with sec-list on the Endpoint
Step 4 : Either use your customized list or use the ones which i have provided in the above section
Step 5 : Then simply start the attack, Start checking for 200 status
Step 7 : Once their is 200 status OK, Start the recursive scan on the same endpoint for juicy information like swagger doc and so on.
step 8 : Other method is to change the API version and try bruteforcing the same endpoint
Eg: Redacted.com/api/v1/{Endpoint} ----- Redacted.com/api/v2/{Endpoint}
```
* Note: Their will be minimum limits per request which will be assigned without API keys so make sure to utilize manual approach as much as you can,Then the rest can be automated for scanning the vulnerability in API with automated tools.

## Contributor:
- [N3T_hunt3r](https://twitter.com/N3T_hunt3r)
