# Easy CVEs using Research

### Tools

* `Google`
* `Twitter`
* `Nuclei`

---

### Steps:

1. **Grab all the subdomains**:

   ```bash
   subfinder -d domain.com -o subs.txt
   ```

2. **Grab all alive domains**:

   ```bash
   httpx -l subs.txt -mc 200 -o alive.txt
   ```

3. **Run Nuclei scans separately** for different template categories and store each result in a different file:

   ```bash
   nuclei -l alive.txt -t nuclei-templates/http/misconfiguration -o misconfigurations.txt
   nuclei -l alive.txt -t nuclei-templates/http/exposed-panels -o exposed-panels.txt
   nuclei -l alive.txt -t nuclei-templates/http/cves -o cves.txt
   nuclei -l alive.txt -t nuclei-templates/http/technologies -o technologies.txt
   ```


4. **Read each output carefully** with patience.

5. **Find interesting tech** used by target (e.g. Jira, WordPress, etc.).

6. **Visit the page** and check the version used.

7. **Google search** with that version like:
   `jira <version> exploit`

8. **Grep CVE IDs** that look promising.

9. **Search CVE on Twitter** (`CVE-XXXX-XXXX poc` or `CVE-XXXX-XXXX exploit`)

10. **Google the CVE or exploit keywords** for better PoCs or writeups.

11. **Test all CVEs** — if successful, report it! 

---

### Authors

* [@Virdoex\_hunter](https://x.com/Virdoex_hunter)
* [@Psikoz\_Coder](https://x.com/Psikoz_Coder)
