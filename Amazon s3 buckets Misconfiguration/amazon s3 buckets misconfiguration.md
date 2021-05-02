### How to check for amazon s3 bucket misconfiguration
````
First of all, you need to install aws cli -

 * Pip install awscli
 *  Set access key 
 * Set secret key

aws s3 ls s3://<bucket name>

See if you are able to access that bucket.

If the bucket is not accessible, still we can try to exploit it.

If you are getting some errors then run this command 
aws s3 ls s3://<bucket name> --no-sign-request

Try moving the files or deleting it and see if you are able to do that or not 
If it can be moved then it is vulnerable and you can report it otherwise it is not vulnerable

For example -

Make a file 
echo "Testing purpose" >> test.txt 

Command to move the file into the bucket. 

aws s3 mv test.txt s3://<bucket name>

Command to delete the file into the bucket

aws s3 rm test.txt s3://<bucket name>/test.txt (if that is present)

````
### References :

 * [Hackerone Report](https://hackerone.com/reports/700051)
 * [Hackerone Report](https://hackerone.com/reports/229690)
 * [https://bugbountypoc.com/s3-bucket-misconfiguration-from-basics-to-pawn/](https://bugbountypoc.com/s3-bucket-misconfiguration-from-basics-to-pawn/)

### Author :
 
 * [Anishka Shukla](https://twitter.com/AnishkaShukla)


