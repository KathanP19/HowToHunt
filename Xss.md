# Methods
Many methods out in wild but here are few most common , but not limited.

### 1. Using Burp
 1. Download Reflection and sentinal plugin for burp.
 2. Walk and spider the target site.
 3. Check the reflected params tab in burp
 4. send that sentinal or check manually.
 
### 2. Using Waybackurls and other similar site
 1. Use [Gau](https://github.com/lc/gau) or [Wayback urls](https://github.com/tomnomnom/waybackurls) to passively gather urls of the target.
 2. Filter the parameters using `grep "="` or gf patterns and store it in a new file.
 3. Now run [kxss](https://github.com/tomnomnom/hacks/tree/master/kxss) or [bxss](https://github.com/ethicalhackingplayground/bxss/) on that new file.
 4. Check Reflected Param Manually or use some tool like [dalfox](https://github.com/hahwul/dalfox) 
