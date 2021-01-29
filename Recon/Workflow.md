## Recon workflow

1. IP space discovery
2. TLDs, Acquisitions, & Relations
3. Subdomain Enum
4. Fingerpirnting
5. Dorking
6. Content Discovery
7. Parameter Discovery

## ASN Discovery

**ASN Discovery of Target:** 

[https://bgp.he.net](https://bgp.he.net/) 

**ASN using whois:** 

`whois -h whois.cymru.com $(dig +short example.com)` 

NOTE: Be careful cause sometimes you might get ASN for VPSs like digital ocean etc. Don't work on them.

**Using Nmap & ASN for discoverying IP related to the targetted ASN**

`nmap --script targets-asn --script-args targets-asn.asn=<ASN Number>`

**Gathering Company intel using AMASS**

`amass intel -org <Organisation name(not domain)>`

**ARIN for ASN:**

[`https://whois.arin.net`](https://whois.arin.net/) 

**Site: IPINFO for ASN**

[`https://ipinfo.io`](https://ipinfo.io/)

**Subdomains using ASNs using AMASS:**

`amass intel -asn <ASN_number>`

## Discovering Brands

-***Looking for acquisition or related orgs to target***

- wikipedia
- Crunchbase

[Crunchbase: Discover innovative companies and the people behind them](https://www.crunchbase.com)

- Owler

[](http://owler.com/)

- Accquiredby

[AcquiredBy | Definitive list of bootstrapped acquisitions](https://acquiredby.co/)

- LinkedIn
- ReverseWhois using amass intel module

    `amass intel -d [domain.com](http://domain.com) -whois`

- BuiltWith

[BuiltWith](https://builtwith.com/)

- Google dork:

`intext:"copyright ©️ org_name"`

- Shodan Dork using HTTP favicon hashes

`http.favicon.hash:<hash>`

**Favicon hash can be found using [favfreak](https://github.com/devanshbatham/FavFreak)**

### Author
[Mr._fr3qu3n533](https://twitter.com/mr_fr3qu3n533)
