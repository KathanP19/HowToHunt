# Take a subdomain , what you can do ...

### 1 - Directory brute-forcing 

    dirsearch -u abc.target.com -w /path/to/wordlist.txt -x 400,404 -o directories.txt

### 2 - Crawl that subdomain 
       
    waybackurls abc.target.com | gau subdomain.target.com | sort -u | httpx -mc 200,301,302 -o all_urls.txt  

### 3 - Filter js files & analyse manually (Personally recommend..!!)

    grep -E "\.(js)$" all_urls.txt > all_js.txt

### 4 - Look for other juicy files

    grep -E "\.(exe|txt|log|cache|secret|db|backup|yml|json|gz|rar|zip|config|py|sql|bak|old|bkp|ini|sh|rb|cgi|jar|key|ovpn|htpasswd|htaccess|dockerfile)$" all_urls.txt > juicy_files.txt 


### 5 - Filter the Endpoints 

    cat all_urls.txt | sed "s/=[^&]*/=/" > all_endpoints.txt

### 6 - Check for parameters by gf(by @tomnomnom)

     cat all_endpoints.txt | gf xss > xss_endpoints.txt
     cat all_endpoints.txt | gf ssrf > ssrf_endpoints.txt
     cat all_endpoints.txt | gf sqli > sqli_endpoints.txt
     cat all_endpoints.txt | gf idors > idors_endpoints.txt
     cat all_endpoints.txt | gf lfi > lfi_endpoints.txt

### 7 - You can also audit that subdomain using burp active scan

    Target --> Scope --> add [abc.target.com] --> sitemap --> Right click on [abc.target.com] --> Actively Scan this host
 
### 8 - Do network scan , it might be possible that they run some hidden services (also use nmap script scan).

    nmap -Pn -vv <abc.target.com>            // you can also use naabu and other tools
    nmap -T5 -Pn -vv -A -p port1,port2,port3 --script vuln <abc.target.com> -oN network.txt

### 9 - Run nuclei , may be you get low hanging bugs or any CVE

    nuclei -u abc.target.com

### NOTE : 
  ###### This basic approach provides a guide for beginners to approach a target for security vulnerabilities. 
  ###### However, you should continue to expand your knowledge and experience with different testing techniques and tools.
