### Implementing the Sign Up Feature:

We will take the example of a School Website(**school.org**) to learn the implementation of Sign Up Feature:  
In this Example, The Students need to register to **school.org** for accessing their Academic educational resource. Users of **school.org** must have the ability to register as a member thus gaining access to the content of the site.

So, The Signup process can be implemented by school in two ways:

1.  **Manual Signup** — Registration based on user providing a series of specific user information. It usually includes form like name, email, password, confirm password, etc. as shown in image below.


2.  **Social Signup** **/OAuth**— Registration via an integrated social media source via social media platform like _Facebook_, _Twitter_, or _Google_, the user can sign into a third party website instead of creating a new account specifically for that website.

In this Blog I will be talking about Bugs in Manual Sign up. Lets have Social Signup/ OAuth for our next blog topic.

### Exploiting Signup Feature:

#### 1\. Duplicate registration / Overwrite existing user.

Duplicate registration is when an application allows us to register or sign up with the same email address, username or phone number. It can have critical consequences based on what kind of attack is performed.

**_Steps to reproduce:_**

1) Create first account in application with email say [abc@gmail.com](mailto:abc@gmail.com) and password.  
2) Logout of the account and create another account with same email and different password.  
3) You can even try to change email case in some case like from [abc@gmail.com](mailto:abc@gmail.com) to [Abc@gmail.com](mailto:Abc@gmail.com)  
4) Finish the creation process — and see that it succeeds  
5) Now go back and try to login with email and the new password. You are successfully logged in.

> **Further Read**  
>  [https://hackerone.com/reports/187714](https://hackerone.com/reports/187714)  
>  [https://shahjerry33.medium.com/duplicate-registration-the-twinning-twins-883dfee59eaf](https://shahjerry33.medium.com/duplicate-registration-the-twinning-twins-883dfee59eaf)  
>  [https://blog.securitybreached.org/2020/01/22/user-account-takeover-via-signup-feature-bug-bounty-poc/](https://blog.securitybreached.org/2020/01/22/user-account-takeover-via-signup-feature-bug-bounty-poc/)

#### 2\. DOS at Name/Password field in Signup Page.

By sending a very long string (100000 characters) it’s possible to cause a denial a service attack on the server. This may lead to the website becoming unavailable or unresponsive. Usually this problem is caused by a vulnerable string hashing implementation. When a long string is sent, the string hashing process will result in CPU and memory exhaustion.

**_Steps to reproduce:_**

1) Go Sign up form.  
2) Fill the form and enter a long string in password  
3) Click on enter and you’ll get 500 Internal Server error if it is vulnerable.

> Further Read  
>  [https://shahjerry33.medium.com/long-string-dos-6ba8ceab3aa0](https://shahjerry33.medium.com/long-string-dos-6ba8ceab3aa0)  
>  [https://hackerone.com/reports/738569](https://hackerone.com/reports/738569)  
>  [https://hackerone.com/reports/223854](https://hackerone.com/reports/223854)

#### 3\. Cross-Site Scripting (XSS) in username, account name for registration.

**Cross-site Scripting** (**XSS**) is a security vulnerability usually found in websites and/or web applications that accept user input. This injects the malicious code into the targeted website’s content, making it a part of the website and thus allowing it to affect victims who may visit or view that website.

Now, for testing Signup page for XSS we can simply insert XSS payoad in fields like: username, email, password,etc.

Payload for Username field : **<svg/onload=confirm(1)>**  
Payload for Email field : **“><svg/onload=confirm(1)>”@x.y**

> Further Read  
>  [https://hackerone.com/reports/196989](https://hackerone.com/reports/196989)  
>  [https://hackerone.com/reports/470206](https://hackerone.com/reports/470206)  
>  [https://hackerone.com/reports/119090](https://hackerone.com/reports/119090)

#### 4\. No Rate Limit at Signup Page.

A **rate limiting** algorithm is used to check if the user session (or IP address) has to be **limited** based on the information in the session cache. Testing for Rate limit at Signup page is quite a good idea.

The Impact can be explained very well. If there is no rate limiting on signup page a malicious users can generate hundreds and thousands of fake accounts that lead to fill the application DataBase with fake accounts, Which can impact the business in many ways.

You can easily test for it with Burp Intruder.  
1\. Capture the signup request and send it to Intruder.  
2\. Add different emails as payload .  
3\. Fire up Intruder, And check whether it returns 200 OK.


> Further Read  
>  [https://hackerone.com/reports/905692](https://hackerone.com/reports/905692)  
>  [https://hackerone.com/reports/97609](https://hackerone.com/reports/97609)  
>  [https://hackerone.com/reports/262830](https://hackerone.com/reports/262830)

#### 5\. Insufficient Email Verification.

Insufficient Email Verification means the application doesn’t verify the email id or the verification mechanism is too weak to be bypassed. You can easily Bypass Email Verification with some of the following common methods like:

1.  Forced Browsing. (directly navigating to files which comes after verifying the email)
2.  Response or Status Code Manipulation. (Replacing the bad response status like 403 to 200 can be useful)
3.  There are much more ways of bypassing . **Tip**: Just google it.

> Further Read  
>  [https://hackerone.com/reports/1040047](https://hackerone.com/reports/1040047)  
>  [https://hackerone.com/reports/617896](https://hackerone.com/reports/617896)  
>  [https://hackerone.com/reports/737169](https://hackerone.com/reports/737169)


**_Thanks for Reading. Any Suggestions are always welcomed!!_**

Follow me for more….

Twitter: [https://twitter.com/Sm4rty\_](https://twitter.com/Sm4rty_)  
LinkedIn: [https://www.linkedin.com/in/sm4rty](https://www.linkedin.com/in/sm4rty)  
Instragram: [https://www.instagram.com/sm4rty](https://www.instagram.com/sm4rty)
