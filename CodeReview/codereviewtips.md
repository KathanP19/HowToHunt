# Code review:-

by performing source code review we can find some web application vulnerabilities


1.Important functions first
------------------------------------
When reading source code, 
focus on important functions such as authentication, password reset, state-changing actions and sensitive info reads. 
(What is the most important would depend on the application.) 
Then, review how these components interact with other functionality.
 Finally, audit other less sensitive parts of the application.

2.Follow user input
------------------------------

Another approach is to follow the code that processes user input. 
User input such as HTTP request parameters, HTTP headers, HTTP request paths, database entries, file reads, and 
file uploads provide the entry points for attackers to exploit the application’s vulnerabilities.This may also help us to
find some critical vulnerabilities like xxe,xxs,sql injection

3.Hardcoded secrets and credentials: 
-------------------------------------------------------
Hardcoded secrets such as API keys, encryption keys and database passwords can be easily discovered during a 
source code review. You can grep for keywords such as “key”, “secret”, “password”, “encrypt” or regex search 
for hex or base64 strings (depending on the key format in use).

4.Use of dangerous functions and outdated dependencies: 
----------------------------------------------------------------------------------
Unchecked use of dangerous functions and outdated dependencies are a huge source of bugs.
 Grep for specific functions for the language you are using and search through the dependency versions list to 
see if they are outdated.

5.Developer comments, hidden debug functionalities, configuration files, and the .git directory:
-----------------------------------------------------------------------------------------------------------------------
 These are things that developers often forget about and they leave the application in a dangerous state. 
Developer comments can point out obvious programming mistakes, hidden debug functionalities often lead to
 privilege escalation, config files allow attackers to gather more information about your infrastructure and finally, 
an exposed .git directory allows attackers to reconstruct your source code.

6.Hidden paths, deprecated endpoints, and endpoints in development:
-----------------------------------------------------------------------------------------------------
 These are endpoints that users might not encounter when using the application normally. But if they work and 
they are discovered by an attacker, it can lead to vulnerabilities such as authentication bypass and sensitive 
information leak, depending on the exposed endpoint.



7.Weak cryptography or hashing algorithms: 
-----------------------------------------------------------------------------------------------------------------------
This is an issue that is hard to find during a black-box test, but easy to spot when reviewing source code. 
Look for issues such as weak encryption keys, breakable encryption algorithms, and weak hashing algorithms. 
Grep for terms like ECB, MD4, and MD5.

8.Missing security checks on user input and regex strength:
-----------------------------------------------------------------------------------------------------
Reviewing source code is a great way to find out what kind of security checks are missing. 
Read through the application’s documentation and test all the edge cases that you can think of. 
A great resource for what kind of edge cases that you should consider is PayloadsAllTheThings.(github)

9.Missing cookie flags:
----------------------------------------------------------------- 
Look out for missing cookie flags such as httpOnly and secure.


10.Unexpected behavior, conditionals, unnecessarily complex and verbose functions: 
--------------------------------------------------------------------------------------------------------------------
Additionally, pay special attention to the application’s unexpected behavior, conditionals, and complex functions. 
These locations are where obscure bugs are often discovered.

 

