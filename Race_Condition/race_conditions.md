# RACE CONDITIONS
## What is Race conditions ? 
 - Race conditions are a common type of vulnerability closely related to business [logic flaws](https://portswigger.net/web-security/logic-flaws). 
 - They occur when websites process requests concurrently without verifying. This can lead to multiple distinct threads interacting with the same data at the same time, resulting in a "collision" that causes unintended behavior in the application.

 <hr>
 
## Limit over run RC (Exploiting Logic Flaws) 
- There are some basic RC tests that you can try in the context of Logic Flaws.
  - Invite user
  - Joining a group
  - Like, subscribe, follow, unfollow, Vote ..etc that required limit. 

### This method required Burp version 2023.9.x or higher (This is the easiest method to exploit, you can create your own script also.)
1 - Send the request to repeater for `'n' no. of times`.
2 - Create a Tab for all those request and choose `Send Parallel (single Packet Attack)`
3 - Hit send , if application is Vulnerable, you'll see the magic.
<hr>

## Rate-Limit Bypass via RC

1 - Select the parameter in request that you want to bruteforce(let's say password), and `send the request into TurboIntruder`.
2 - If it is password or something , then wordlist should be copied in your clipboard that and use the below python script in Turbo Intruder.

```
def queueRequests(target, wordlists):

    # as the target supports HTTP/2, use engine=Engine.BURP2 and concurrentConnections=1 for a single-packet attack
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=1,
                           engine=Engine.BURP2
                           )
    
    # assign the list of candidate passwords from your clipboard
    passwords = wordlists.clipboard
    
    # queue a login request using each password from the wordlist
    # the 'gate' argument withholds the final part of each request until engine.openGate() is invoked
    for password in passwords:
        engine.queue(target.req, password, gate='1')
    
    # once every request has been queued
    # invoke engine.openGate() to send all requests in the given gate simultaneously
    engine.openGate('1')

def handleResponse(req, interesting):
    table.add(req)
```

3 - Hit Attack and see the magic.

<hr>

## Multi-Endpoint Race Conditions

1 - When a single functionality chains with multiple request , eg - `Buying a product from a E-Commerce application`

    - /product --> for the product
    - /cart    --> Add to cart that product
    - /cart/checkout  --> Buy that product

2 - Send all the required request into burp repeater for the product you want in a sequence and create a Tab.
3 - Select `Send Parallel (single Packet Attack)` and hit send.

<hr>

##  Single Endpoint RaceCondition

 - If you've facing a functionality where new objects edit the older object and require email verification, then we can test there for RaceConditions, Eg. Email Change functionality

       `Account A : Attacker --> attacker@email.com` 
       `Account B : Victim --> victim@email.com`

1 - Application has email change functionality, where the new requested email is updated over older email, and send the confirmation link to the user email address.
  
2 - Since, email is updated in DataBase and only confirmation is needed, 
   
3 - So we `Send Parallel (Single Packet Attack)` of the changing email for, 
   
      `attacker@email.com`
      `victim@email.com`
 
4 - In Backend, Because we request so much fast that when application server generate confirmation link for `attacker@email.com` at the same time `victim@email.com` request is also reach their and application got confused to prioritise , As a result it sends both confirmation links on the same email.

5 - Impact : This will lead to Full Acount Takeover.

<hr>

## Time Sensitive Vulnerabilities
1 - Send two parallel `forget password` request for `Attacker i..e Account-A`

2 - If both `password reset links contains same token`, then we can test their for ATO.

3 - This time send both request again by `changing victim's username/password in one of them`.

4 - Analyze the response time, if `both request have same response time`, then their might be chances of ATO

<hr><hr>

# REAL World Cases : (H1 reports)

### 1 - Race condition in flag submission
   - [Found in --> HackerOne](https://hackerone.com/reports/454949)
   - Report describes a Race Condition Vulnerability which `allow an authenticated user to submit the same ctf Flag multiple times`. Increasing the user points and therefore the chances to get an invitation to a private program.

### 2 - Race condition on Invite user action
   - [Found in --> Omise](https://hackerone.com/reports/1285538)
   - Race condition vulnerability which `allows the invitation of the same member multiple times to a single team` via the dashboard.

### 3 - Race condition in performing retest allows duplicated payments
   - [Found in --> HackerOne](https://hackerone.com/reports/429026)
   - By executing `multiple requests to confirm a retest at the same time`, a malicious user is paid multiple times for the retest. This `allows for stealing money from HackerOne`, which could go unnoticed by both HackerOne and the attacker.

### 4 - Race Condition leads to Un-Deletable group member
   - [Found in --> HackerOne](https://hackerone.com/reports/604534)
   - Small Race condition bug in which a group `user couldn't be removed from the group even by the admin` after they join.

### 5 - Race Condition when following a user
   - [Found in --> every.org](https://hackerone.com/reports/927384)
   - Race condition vulnerability when following a user. If you send the Follow requests asynchronously, you can `follow a user multiple times instead getting an error message`.

### 6 - Race Conditions in Popular reports feature.
   - [Found in --> HackerOne](https://hackerone.com/reports/146845)
   - This report describes a race condition bug which `allow an authenticated user to upvote or downvote multiple times a single report`, increasing its counter (and its rank on the hacktivity page).

### 7 - Race condition in joining CTF group
   - [Found in --> HackerOne](https://hackerone.com/reports/1540969)
   - A race condition in https://ctf.hacker101.com/group/join `allows a user to join the same CTF group multiple times`.The user will show up in the group member list multiple times, and affect the group statistics.

### 8 - Race conditions can be used to bypass invitation limit
   - [Found in --> KeyBase](https://hackerone.com/reports/115007)
   - Using Race conditions, attacker was `able to send out a total of 7 invites to his throwaway emails, obviously bypassing the 3 no. of invitations limit`.

### 9 - Race Condition allows to redeem multiple times gift cards.
   - [Found in --> Reverb.com](https://hackerone.com/reports/759247)
   - I've found a Race Condition vulnerability which `allows to redeem gift cards multiple times`. This how an attacker can easily buy stuff just buying one gift card and redeem it over and over again.
