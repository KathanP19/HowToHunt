# Subdomain Enumeration 
Well, subdomain enumeration is important when you are hunting on wildcard enable scope programs. 
If you are able to get unique subdomains that other miss then it's a good chance for you to get some bugs

# General Methodology
* Passive
* Active
* Permutation

## Passive
In this stage you have to use as much resources as you can to passivly gather subdomains
Now a days it's not that much hard to do with community standard tools that usages API keys

### Tools

* Subfinder
* Amass
* Assetfinder
* Findomain

## Active
In this stage you have to perform bruteforcing on your target host to see if the word from your wordlist resolve as valid subdomain or not

### Tools

* ShuffleDNS
* Aiodnsbrute

## Permutation
In this stage you have to play around the subdomains. Now do changed with the words and see still it resolve as valid or not

## Portscan
Convert domains into ip address
```bash
while read l; do ip=$(dig +short $l|grep -oE "\b([0-9]{1,3}\.){3}[0-9]{1,3}\b"|head -1);echo "[+] '$l' => $ip";echo $ip >> ips.txt;done < domains.txt

```

we will use masscan for faster results

>masscan -p1-65535 -iL ips.txt --max-rate 1800 -oG output.log

or you can use [Naabu](https://github.com/projectdiscovery/naabu), [RustScan](https://github.com/RustScan/RustScan/).

### Tools

* AltDNS
* DNSGen + ShuffleDNS

## Reference & Resources

https://pentester.land/cheatsheets/2018/11/14/subdomains-enumeration-cheatsheet.html

https://0xpatrik.com/subdomain-enumeration-2019/

https://0xpatrik.com/subdomain-enumeration-smarter/

https://rootsploit.com/bug-bounty-recon-faster-port-scan/

Theres a lot you can do. For now just mentioning communty standard approaches. Will be updating it regularly depending on the methodology comes out. 

## Framework
An automated framework can be used to automate those whole workflow

* [SEF](https://github.com/remonsec/SEF)
___
## Author
[Mehedi Hasan Remon](https://twitter.com/remonsec)
[Rishi Choudhary](https://twitter.com/0xRyuk)