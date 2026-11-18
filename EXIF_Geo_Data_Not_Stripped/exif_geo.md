<h4>Summary:</h4>
When a user uploads an image in example.com, the uploaded image’s EXIF Geolocation Data does not gets stripped. As a result, anyone can get sensitive information of example.com users like their Geolocation, their Device information like Device Name, Version, Software & Software version used etc.

<h4>Steps to reproduce:</h4>

1. Got to Github ( https://github.com/ianare/exif-samples/tree/master/jpg) <br>
2. There are lot of images having resolutions (i.e 1280 * 720 ) , and also whith different MB’s . <br>
3. Go to Upload option on the website <br>
4. Upload the image<br>
5. Capture the URL of the image served by the application, then download that processed copy.</br>
6. Open the downloaded copy in [Metadata Viewer](https://metadataremover.ai/metadata-viewer) to inspect its EXIF, GPS, IPTC, and XMP fields.</br>
7. Check whether sensitive metadata remains. Preserve the original sample and confirm important values with a second parser such as [ExifTool](https://exiftool.org/) before reporting.

# Reports (Hackerone)

- [IDOR with Geolocation data not stripped from images](https://hackerone.com/reports/906907)

# Author
* [@0xd3vil](https://twitter.com/0xd3vil)
* [@klaus](https://twitter.com/klaus_dev)
