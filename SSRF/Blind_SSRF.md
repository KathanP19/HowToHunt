# Blind SSRF
Blind SSRF's are those that don't show enumerated data directly to the user and hence are known as blind SSRF.

## Different Methods:

### Methodology #1:
**Header** **Injection**:

One way of finding them is by inserting your burp collaborator domain into the referrer header also known as host header injection.

Snippet:
```
    GET /HTTP 1.1
    Host: site.tld
    User Agent: Firefox
    Referrer: https://your_collaborator_instance.com

```


 Many organizations use services that analyse which url or service is referring the visitor to their site. Execution of this type of attack depends upon the underlying service in my case the server was running on an aws ec2 instance but i was unable to get to it's admin panel namely (192.168.192.168) as it was only performing a lookup on me but not allowing anythng beyond that. Try it on different sites and services that you come across you just might get lucky.

I will list more as i find if you have found any please kindly list them here so that other's beneift from it.

### Contributor:
 * [@cowlingbanana](https://github.com/cowlingbanana)

