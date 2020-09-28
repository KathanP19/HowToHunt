# Bypassing File Uploads

Suppose you have a limitation that you can only upload in a few formats like PDF, JPEG, JPG, ….But what if you can upload a PHP file by defying the Upload mechnism and validation of file type check. let me tell you if someone can upload a PHP file then its game over for the website as he will upload a php shell and can easily perform an RCE , or Worst will simply gain a reverse shell on the server.

> __How does Bypass work__

Well it depends on which kind of validation the system is using …it is just verfying the extension ?? if its just doing that then it becomes very easy to bypass and upload a PHP file or something malicious. suppose we have to upload a JPG file so the extension must be something.jpg

### 1. Bypassing Normal extension
Now what we can do is we can upload a file which looks like this something.php.jpg or somethings.jpg.php.
### 2. Bypassing the magic Byte validation.

For this method we use polygots. Polyglots, in a security context, are files that are a valid form of multiple different file types. For example, a GIFAR is both a GIF and a RAR file. There are also files out there that can be both GIF and JS, both PPT and JS, etc.

so while we have to upload a JPEG file type we actaully can upload a PHAR-JPEG file which will appear to be a JPEg file type to the server while validating. the reason is the file PHAR-JPEg file has both the JPEG header and the PHP file also. so while uploading it didn’t get detected and later after processing the PHP file can be used to exploit.

And at last Uploading a shell to some random websites for fun is not really cool so don’t ever try untill unless you have the permission to test.

-----

### Author 

- [harsha](https://twitter.com/harsha0x01)
