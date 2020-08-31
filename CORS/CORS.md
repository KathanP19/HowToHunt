# Misconfigured CORS
 Here are few methods and steps you can do to check for misconfigure cors.

* Hunting method 1(Single target):

```
Step->1. Capture the target website and spider or crawl all the website using burp.
Step->2. Use burp search look for Access-Control
Step->3. Try to add Origin Header i.e,Origin:attacker.com or Origin:null or Origin:attacker.target.com or Origin:target.attacker.com
Step->4  If origin is reflected in response means the target is vuln to CORS
```
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------

* Hunting method 2(mutliple means including subdomains):
```
step 1-> find domains i.e subfinder -d target.com -o domains.txt
step 2-> check alive ones : cat domains.txt | httpx | tee -a alive.txt
step 3-> send each alive domain into burp i.e, cat alive.txt | parallel -j 10 curl --proxy "http://127.0.0.1:8080" -sk 2>/dev/null
step 4-> Repeat hunting method 1
```
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------

* Both above method are manual methods so lets check an automated way
# Tools
* [https://github.com/chenjj/CORScanner](https://github.com/chenjj/CORScanner)
* [https://github.com/lc/theftfuzzer](https://github.com/lc/theftfuzzer)
* [https://github.com/s0md3v/Corsy](https://github.com/s0md3v/Corsy)
* [https://github.com/Shivangx01b/CorsMe](https://github.com/Shivangx01b/CorsMe)

# Automate Way :
```
step1-> find domains i.e, subfinder -d domain.com -o target.txt
step2-> grep alive: cat target.txt | httpx | tee -a alive.txt
step3-> grep all urls using waybackurls by @tomnomnom and gau tool i.e,cat alive.txt | gau | tee -a urls.txt
step4-> run any of these tools on each url 
step5-> configure the manually
```
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Another Method

### Tools You Will Need for this method.
* [https://github.com/tomnomnom/meg](https://github.com/tomnomnom/meg)
* [https://github.com/tomnomnom/gf](https://github.com/tomnomnom/gf)
* [https://github.com/projectdiscovery/subfinder](https://github.com/projectdiscovery/subfinder)
* [https://github.com/tomnomnom/assetfinder](https://github.com/tomnomnom/assetfinder)
* [https://github.com/Edu4rdSHL/findomain](https://github.com/Edu4rdSHL/findomain)
* [https://github.com/projectdiscovery/httpx](https://github.com/projectdiscovery/httpx)
         
### Steps
```
1) Find Domains with the help of subfinder,assetfinder,findomain i.e , subfinder -d target.com | tee -a hosts1 , findomain -t target.com | tee -a hosts1 , assetfinder --subs-only target.com |tee -a hosts1 .
2) Then cat hosts1 | sort -u | tee -a hosts2 and then cat hosts2 | httpx | tee -a hosts .
3) Navigate through terminal where hosts file is located  echo "/" > paths
4) Then type meg -v
5) After the completion of process type gf cors.
6) All the urls with Access-Control-Allow will be displayed.  
```

# Authors
* [@Virdoex_hunter](https://twitter.com/Virdoex_hunter)
