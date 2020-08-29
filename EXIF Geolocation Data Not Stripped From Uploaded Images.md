<h4>Summary:</h4>
When a user uploads an image in example.com, the uploaded image’s EXIF Geolocation Data does not gets stripped. As a result, anyone can get sensitive information of example.com users like their Geolocation, their Device information like Device Name, Version, Software & Software version used etc.

<h4>Steps to reproduce:</h4>
1. Got to Github ( https://github.com/ianare/exif-samples/tree/master/jpg)
2. There are lot of images having resolutions (i.e 1280 * 720 ) , and also whith different MB’s .
3. Go to Upload option on the website
2. Upload the image
3. see the path of uploaded image ( Either by right click on image then copy image address OR right click, inspect the image, the URL will come in the inspect ,   edit it as html )
4. open it (http://exif.regex.info/exif.cgi)
5. See wheather is that still showing exif data , if it is then Report it.

