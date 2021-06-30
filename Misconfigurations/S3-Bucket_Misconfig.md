
##  How to check for amazon S3 Bucket Misconfiguration.
* First of all, you need to install aws cli - `Pip install awscli`
* Dont Forget to Set:
\- Access key 
\- Secret key

**1.** **Check is you can list iteams from the bucket.** 
`aws s3 ls s3://<bucket name>`
* See if you are able to access that bucket.
* If the bucket is not accessible, still we can try to exploit it.

* If you are getting some errors then run this command 
`aws s3 ls s3://<bucket name> --no-sign-request`

**2. Try moving the files or deleting it and see if you are able to do that or not** 
* If it is possible to move files then it is vulnerable and you can report it otherwise it is not vulnerable
* First Make a file 
`echo "Testing purpose" >> test.txt `
* Now try command to move the file into the bucket. 
`aws s3 mv test.txt s3://<bucket name>`
* Also try command to copy the file from local drive to the S3 bucket. 
`aws s3 cp test.txt s3://[bucketname]/test.txt`

**3.  Delete files from the bucket.**
* Command to delete the file into the bucket
`aws s3 rm test.txt s3://<bucket name>/test.txt` *(if that is present)*


## References :
 * [Hackerone Report](https://hackerone.com/reports/700051) 
 * [Hackerone Report](https://hackerone.com/reports/229690) 
 * [https://bugbountypoc.com/s3-bucket-misconfiguration-from-basics-to-pawn](https://bugbountypoc.com/s3-bucket-misconfiguration-from-basics-to-pawn)
 
## Author :
 * [Anishka Shukla](https://twitter.com/AnishkaShukla)
 * [Anubhav Singh](https://twitter.com/AnubhavSingh_)
