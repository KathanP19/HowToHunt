# Finding Origin IPs Behind WAFs

## Introduction

Web Application Firewalls (WAFs) like Cloudflare, AWS WAF, and others protect web applications by filtering and monitoring HTTP traffic. However, discovering the origin IP address behind these protective layers can be crucial during security assessments. This guide outlines various techniques to identify origin IPs.

## Identifying the Presence of a WAF

Before attempting to bypass a WAF, first confirm its presence:

```bash
# Get the IP address
dig +short example.com

# Check the organization
curl -s https://ipinfo.io/IP | jq -r '.org'
```

**Common WAF Indicators:**
- AWS WAF: Look for "AWSLB" and "AWSLBCORS" cookies
- Cloudflare: Organization info will indicate Cloudflare, Inc.
- Other WAFs may have specific signatures or response headers

## Techniques for Origin IP Discovery

### 1. Historical DNS Records

Historical DNS records often reveal IPs used before WAF implementation:

- **SecurityTrails DNS History**
  - Visit: https://securitytrails.com/domain/example.com/dns
  - Export historical A records
  - Extract IPs:
    ```bash
    grep -E -o "([0-9]{1,3}[\\.]){3}[0-9]{1,3}" dns_history.txt | sort -u > potential_ips.txt
    ```

- **DNS Dumpster**
  - Use https://dnsdumpster.com to generate network maps
  - Look for non-WAF IP addresses in the results

### 2. Subdomain Enumeration

Development or staging environments often lack proper WAF protection:

```bash
# Find subdomains and their IPs
subfinder -silent -d example.com | dnsprobe -silent | awk '{ print $2 }' | sort -u > subdomain_ips.txt
```

Focus on subdomains like:
- dev.example.com
- staging.example.com
- test.example.com
- beta.example.com

### 3. SSL Certificate Information

Certificates often reveal all domains and IPs where they're deployed:

- **Censys Method**:
  1. Search for certificates using your target domain
  2. Select "Certificates" in the input field and search for your domain
  3. Review each certificate and click "Explore" > "IPv4 Hosts"
  4. Collect all associated IPs

- **Shodan Method**:
  ```
  # Search by Common Name (CN)
  ssl.cert.subject.CN:"example.com"
  
  # Search in all certificate fields (broader)
  ssl:"example.com"
  ```

  **Note:** Verify results manually as they may include CDN/proxy IPs. SAN (Subject Alternative Name) fields are generally more reliable than CN.

### 4. Direct IP Testing

For each potential IP, test if it responds to the target hostname:

```bash
# Test single IP
curl -s -k -H "Host: example.com" https://POTENTIAL_IP/

# Test multiple IPs
for ip in $(cat potential_ips.txt); do
  org=$(curl -s https://ipinfo.io/$ip | jq -r '.org')
  title=$(timeout 2 curl -s -k -H "Host: example.com" https://$ip/ | pup 'title text{}')
  echo "IP: $ip | Title: $title | Org: $org"
done
```

### 5. Email Headers Analysis

Emails from the target domain often contain internal IP information:

1. Trigger emails from the target (register, password reset, newsletters)
2. Examine email headers, particularly:
   - Return-Path
   - Received
   - X-Originating-IP

### 6. Specialized Tools

Several tools automate origin IP discovery:

- **CloudFail**:
  ```bash
  git clone https://github.com/m0rtem/CloudFail.git
  cd CloudFail
  pip install -r requirements.txt
  python3 cloudfail.py -t example.com
  ```

- **CloudFlair**:
  ```bash
  git clone https://github.com/christophetd/CloudFlair
  cd CloudFlair
  pip install -r requirements.txt
  python3 cloudflair.py example.com
  ```

## Verifying the Origin IP

After discovering potential origin IPs, verify them:

1. Compare response content with the WAF-protected site
2. Look for server fingerprints (headers, error pages)
3. Check for administrative interfaces or panels not accessible via WAF

## Best Practices

- Combine multiple techniques for better results
- Document all discovered IPs and their verification status
- Check IP ranges belonging to the organization
- Consider timing your requests to avoid rate limiting

## References

- [Navisec: A Pentester's Guide - Unmasking WAFs and Finding the Source](https://delta.navisec.io/a-pentesters-guide-part-5-unmasking-wafs-and-finding-the-source/)
- [Detectify: Bypassing Cloudflare WAF with the Origin Server IP Address](https://blog.detectify.com/2019/07/31/bypassing-cloudflare-waf-with-the-origin-server-ip-address/)

## Credits

### Original Author
* [maverickNerd](https://x.com/maverickNerd)

### Contributors
* [nagarajcruze](https://github.com/nagarajcruze)
* [www](https://github.com/www)

---
*Enhanced and reformatted for HowToHunt repository by [remonsec](https://x.com/remonsec)*
