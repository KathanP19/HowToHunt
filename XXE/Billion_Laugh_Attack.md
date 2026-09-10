# Billion Laughs Attack (XML Entity Expansion DoS)

The Billion Laughs Attack is a denial-of-service variant related to XML parsing: nested entities resolve each other cyclically, so a tiny document expands to billions of characters, consuming excessive CPU/memory and knocking the parser (and often the service) offline. It requires no external network access — only an XML parser without entity/expansion limits. Impact is availability-based (application-level DoS); severity depends on how persistently the payload can be stored/re-parsed.

For classic XXE file-read/blind-exfil hunting, see [XXE Methods](./XXE_Methods.md).

## Hunting Checklist

1. Confirm the endpoint parses XML (see the content-type switching steps in [XXE Methods](./XXE_Methods.md)).
2. Send a *small* expansion first (e.g., up to `&lol4;`) and measure response time/memory — never start with the full `lol9` on production.
3. Escalate expansion depth only if the parser resolves nested internal entities at all.
4. Test stored re-parsing paths (profiles, saved documents, feed items): a payload stored once is re-processed on every future view — a persistent DoS.
5. Verify impact safely against a self-owned feature or with explicit program scope; abort if the service degrades for real users.

## Test Cases & Examples

Reproduction steps:

1. Capture the request in Burp.
2. Send it to Repeater and convert the body to XML whether the endpoint accepts it or not.
3. To confirm, check the `Accept` header — change it to `application/xml` (try `application/json` responses as a fallback probe).
4. Convert JSON to XML if that's the only accepted format.
5. Place the payload inside the XML field and tune the DoS variation by changing `lol1` → `lol9` depending on the desired expansion depth.

Billion Laugh payload:

```xml
<?xml version="1.0"?>
<!DOCTYPE lolz [
 <!ENTITY lol "lol">
 <!ELEMENT lolz (#PCDATA)>
 <!ENTITY lol1 "&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;">
 <!ENTITY lol2 "&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;">
 <!ENTITY lol3 "&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;">
 <!ENTITY lol4 "&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;">
 <!ENTITY lol5 "&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;">
 <!ENTITY lol6 "&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;">
 <!ENTITY lol7 "&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;">
 <!ENTITY lol8 "&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;">
 <!ENTITY lol9 "&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;">
]>
<lolz>&lol9;</lolz>
```

Each level multiplies by 10: `lol9` expands to 10^9 copies of `"lol"` ≈ 3 GB of string data from a ~1 KB request.

Related quadratic-blowup idea (same class): two mutually-recursive entities grow exponentially with document size — same detection applies, so test conservatively.

## Tools & Resources

- Burp Suite Repeater (with "grep response" timing comparison)
- PortSwigger — XML injection / entity expansion lab track: https://portswigger.net/web-security/xxe
- PayloadsAllTheThings — XXE (entity expansion section): https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XXE%20Injection
- OWASP prevention cheat sheet: https://cheatsheetseries.owasp.org/cheatsheets/XML_External_Entity_Prevention_Cheat_Sheet.html

## Report Tips

- Evidence = latency/memory delta between `lol4`-scale and `lol9`-scale requests (or the parser hanging), plus confirmation the request was *parsed*, not just echoed.
- Persistent (stored) variants are more severe: note how many re-parses occur per view of the object.
- Always disclose that testing was throttled to avoid real outage; triagers push back on DoS reports that look reckless.

---
*Contributor: [N3T_hunt3r](https://twitter.com/N3T_hunt3r)*
