# XML External Entities.
These are my methods to check and hunt for XML External Entities.
I might be missing a lot of things but as the community believe in "sharing is caring" by @CXVVMVII.

## Methods
1. Convert the content type from "application/json"/"application/x-www-form-urlencoded" to "applcation/xml".
2. File Uploads allows for docx/xlcs/pdf/zip , unzip the package and add your evil xml code into the xml files.
3. If svg allowed in picture upload , you can inject xml in svgs.
4. If the web app offers RSS feeds , add your milicious code into the RSS.
5. Fuzz for /soap api , some applications still running soap apis
6. If the target web app allows for SSO integration, you can inject your milicious xml code in the SAML request/reponse

## Twitter:
* [whitechaitai](https://twitter.com/whitechaitai)
