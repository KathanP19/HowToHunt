# IDOR

- At its core, an IDOR is an access control vulnerability in which an application relies on user-supplied input to reference objects directly. In this case, the object could be a picture, a comment on a post, personally identifiable information (PII) associated with a user or even an entire department within an organization.
- Insecure Direct Object References occur when an application provides direct access to objects based on user-supplied input. As a result of this vulnerability attackers can bypass authorization and access resources in the system directly, for example database records or files. Insecure Direct Object References allow attackers to bypass authorization and access resources directly by modifying the value of a parameter used to directly point to an object. Such resources can be database entries belonging to other users, files in the system, and more.
- IDORs can exist throughout the entire application so it is always suggested that if you see IDs then to always test, even if they are guids or some type of "encrypted id". Look for potential leaks of this ID (public profile?) or look for patterns and see if you can generate your own & run it through burp intruder.

## Types of IDOR you will see in wild:

1. The value of a parameter is used directly to retrieve a database record
    
    ```markdown
    http://foo.bar/somepage?invoice=12345
    ```
    
2. The value of a parameter is used directly to perform an operation in the system
    
    ```markdown
    http://foo.bar/changepassword?user=someuser
    ```
    
3. The value of a parameter is used directly to retrieve a file system resource
    
    ```markdown
    http://foo.bar/showImage?img=img00011
    ```
    
4. The value of a parameter is used directly to access application functionality
    
    ```markdown
    http://foo.bar/accessPage?menuitem=12
    ```
    

# Testing for IDOR - ( Manual-Method ):

## **Base Steps:**

```markdown
1. Create two accounts if possible or else enumerate users first. 
2. Check if the endpoint is private or public and does it contains any kind of id param.
3. Try changing the param value to some other user and see if does anything to their account.
4. Done !!
```

## Testcase - 1: Add IDs to requests that don’t have them

```jsx
GET /api/MyPictureList → /api/MyPictureList?user_id=<other_user_id>

Pro tip: You can find parameter names to try by deleting or editing other objects and seeing the parameter names used.
```

## Testcase - 2: Try replacing parameter names

```jsx
Instead of this:
GET /api/albums?album_id=<album id>

Try This:
GET /api/albums?account_id=<account id>

Tip: There is a Burp extension called Paramalyzer which will help with this by remembering all the parameters you have passed to a host.
```

## Testcase - 3: Supply multiple values for the same parameter.

```jsx
Instead of this:
GET /api/account?id=<your account id> →

Try this:    
GET /api/account?id=<your account id>&id=<admin's account id>

Tip: This is known as HTTP parameter pollution. Something like this might get you access to the admin’s account
```

## Testcase - 4: Try changing the HTTP request method when testing for IDORs

```jsx
Instead of this:
POST /api/account?id=<your account id> →

Try this:    
PUT /api/account?id=<your account id>

Tip: Try switching POST and PUT and see if you can upload something to another user’s profile. For RESTful services, try changing GET to POST/PUT/DELETE to discover create/update/delete actions.
```

## Testcase - 5: Try changing the request’s content type

```jsx
Instead of this:
```
POST /api/chat/join/123
[…]
Content-type: application/xml → 
<user>test</user>    
```
Try this:
```
POST /api/chat/join/123
[…]
Content-type: application/json
{“user”: “test”}
```

Tip: Access controls may be inconsistently implemented across different content types. Don’t forget to try alternative and less common values like text/xml, text/x-json, and similar.
```

## Testcase - 6: Try changing the requested file type (Test if Ruby)

```jsx
Example:

GET /user_data/2341 --> 401 Unauthorized

GET /user_data/2341.json --> 200 OK

Tip: Experiment by appending different file extensions (e.g. .json, .xml, .config) to the end of requests that reference a document.
```

## Testcase - 7: Does the app ask for non-numeric IDs? Use numeric IDs instead

```jsx
There may be multiple ways of referencing objects in the database and the application only has access controls on one. 
Try numeric IDs anywhere non-numeric IDs are accepted:
Example:

username=user1 → username=1234
account_id=7541A92F-0101-4D1E-BBB0-EB5032FE1686 → account_id=5678
album_id=MyPictures → album_id=12
```

## Testcase - 8: Try using an array

```markdown
If a regular ID replacement isn’t working, try wrapping the ID in an array and see if that does the trick. For example:

{“id”:19} → {“id”:[19]}
```

## Testcase - 9: Wildcard ID

```markdown
These can be very exciting bugs to find in the wild and are so simple. Try replacing an ID with a wildcard. You might get lucky!

GET /api/users/<user_id>/ → GET /api/users/*
```

## Testcase - 10: Pay attention to new features

```markdown
If you stumble upon a newly added feature within the web app, such as the ability to upload a profile picture for an upcoming charity event, and it performs an API call to:

/api/CharityEventFeb2021/user/pp/<ID>

It is possible that the application may not enforce access control for this new feature as strictly as it does for core features.
```

# Testing For IDOR - ( Automated Method ):

[Finding Broken Access Controls](https://threat.tevora.com/finding-broken-access-controls/)

[PimpMyBurp #1 - PwnFox + Autorize: The perfect combo to find IDOR - Global Bug Bounty Platform](https://blog.yeswehack.com/yeswerhackers/pimpmyburp-pwnfox-autorize-find-idor/)

[Automating BURP to find IDORs](https://medium.com/cyberverse/automating-burp-to-find-idors-2b3dbe9fa0b8)

# Reference:

[Everything You Need to Know About IDOR (Insecure Direct Object References)](https://medium.com/@aysebilgegunduz/everything-you-need-to-know-about-idor-insecure-direct-object-references-375f83e03a87)

[Finding more IDORs - Tips and Tricks | Aon](https://www.aon.com/cyber-solutions/aon_cyber_labs/finding-more-idors-tips-and-tricks/)

[KathanP19/HowToHunt](https://github.com/KathanP19/HowToHunt/blob/master/IDOR/IDOR.md)

[Learn about Insecure Object Reference (IDOR) | BugBountyHunter.com](https://www.bugbountyhunter.com/vulnerability/?type=idor)

[WSTG - v4.2](https://owasp.org/www-project-web-security-testing-guide/v42/4-Web_Application_Security_Testing/05-Authorization_Testing/04-Testing_for_Insecure_Direct_Object_References.html)

[IDOR (Insecure Direct Object Reference)](https://notes.mufaddal.info/web/idor)

[What I learnt from reading 220* IDOR bug reports.](https://medium.com/@Sm9l/what-i-learnt-from-reading-220-idor-bug-reports-6efbea44db7)

### Medium:

[Full account takeover worth $1000 Think out of the box](https://mokhansec.medium.com/full-account-takeover-worth-1000-think-out-of-the-box-808f0bdd8ac7)

[All About Getting First Bounty with IDOR](https://infosecwriteups.com/all-about-getting-first-bounty-with-idor-849db2828c8)

[](https://codeburst.io/hunting-insecure-direct-object-reference-vulnerabilities-for-fun-and-profit-part-1-f338c6a52782)

[IDOR that allowed me to takeover any users account.](https://vedanttekale20.medium.com/idor-that-allowed-me-to-takeover-any-users-account-129e55871d8)

[All About IDOR Attacks](https://betterprogramming.pub/all-about-idor-attacks-64c4203b518e)

[Access developer tasks list of any of Facebook Application (GraphQL IDOR)](https://amineaboud.medium.com/access-developer-tasks-list-of-any-of-facebook-application-graphql-idor-62307c5e5b34)

# Tips
```
#1 https://twitter.com/M0_SADAT/status/1361289751597359105
Looking for high impact IDOR?
Always try to find the hidden parameters for this endpoints using Arjun and Parameth
/settings/profile
/user/profile
/user/settings
/account/settings
/username
/profile
And any payment endpoint
```
`Pro tip: Don’t forget to try create/update/delete operations on objects that are publicly readable but shouldn’t be writable. Can you PUT to /api/products and change a price?`

## Author
[KathanP19](https://twitter.com/KathanP19)
