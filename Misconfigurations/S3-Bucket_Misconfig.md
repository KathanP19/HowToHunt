# S3 Bucket Misconfiguration

AWS S3 buckets back application storage, but overly permissive bucket policies or ACLs can allow anyone to list, read, write, or delete objects. Misconfigured buckets leak PII and source code, and writable buckets enable stored-XSS/phishing or full data destruction. Impact ranges from High (data exposure/PII) to Critical (write/delete + sensitive assets).

## Hunting Checklist

1. Install the AWS CLI — `pip install awscli` — and don't forget to set:
   - Access key
   - Secret key
   (a dedicated/no-permission key is fine; many misconfigs are readable with *no* credentials at all).
2. Find candidate bucket names: grep JS bundles, mobile app traffic, source maps and docs for `s3.amazonaws.com` / `s3://` patterns; fuzz common corporate naming (`<company>-backup`, `-static`, `-assets`, `-logs`, `-uploads`).
3. Test **listing** (read exposure):

```bash
aws s3 ls s3://<bucket-name>
# if you get permission errors, try without signing the request:
aws s3 ls s3://<bucket-name> --no-sign-request
```

4. Test **write**: create a local file, then try moving/copying it in:

```bash
echo "Testing purpose" >> test.txt
aws s3 mv test.txt s3://<bucket-name>
aws s3 cp test.txt s3://<bucket-name>/test.txt
```

5. Test **delete** only of your *own* uploaded test object:

```bash
aws s3 rm s3://<bucket-name>/test.txt
```

6. If it is possible to move/copy files in, it is vulnerable and reportable; same for successful deletes.
7. Also check the raw REST surface in the browser and the bucket policy:

```http
GET /?acl HTTP/1.1
Host: <bucket-name>.s3.amazonaws.com

GET /?list-type=2 HTTP/1.1
Host: <bucket-name>.s3.amazonaws.com
```

Note the error difference: `AccessDenied` ⇒ bucket exists (permission issue), `NoSuchBucket` ⇒ free name (possible bucket takeover).

## Test Cases & Examples

```bash
# Enumerate objects anonymously
aws s3 ls s3://<bucket-name> --no-sign-request --recursive

# Upload a harmless marker object
echo "bounty-test-$(date +%s)" > test.txt
aws s3 cp test.txt s3://<bucket-name>/test.txt --no-sign-request

# Confirm then clean up
aws s3 rm s3://<bucket-name>/test.txt --no-sign-request
```

Direct HTTPS verification of a public object:

```http
GET /test.txt HTTP/1.1
Host: <bucket-name>.s3.amazonaws.com
```

## Bypass Techniques

- If `aws s3 ls` fails, retry with `--no-sign-request` — bucket policies that grant `s3:GetObject` to `Principal: "*"` reject *authenticated* mismatches but serve anonymous requests.
- List with explicit regional endpoint when the generic host misbehaves: `aws s3 ls s3://<bucket> --endpoint-url https://s3.<region>.amazonaws.com`.
- Some buckets block `ListBucket` but allow `GetObject` — enumerate object keys from app traffic/JS references and fetch them directly.
- CORS misconfiguration can let a malicious origin read objects even when listing is denied:

```http
OPTIONS / HTTP/1.1
Host: <bucket-name>.s3.amazonaws.com
Origin: https://attacker.example
Access-Control-Request-Method: GET
```

## Tools & Resources

- **awscli** (primary), `trufflehog`/`gau`/JS grep for bucket names, nuclei S3 templates, `BucketLoot` (name permutation generation), S3Scanner / CloudBrute for bulk checking.
- [HackerOne report #700051](https://hackerone.com/reports/700051)
- [HackerOne report #229690](https://hackerone.com/reports/229690)
- [S3 Bucket Misconfiguration — From Basics to Pawn](https://bugbountypoc.com/s3-bucket-misconfiguration-from-basics-to-pawn)

## Report Tips

- Impact framing: listing PII/source = High–Critical (show 1–2 example objects, never bulk-scrape); writable = demonstrate an uploaded non-destructive test file and its URL; deleting *existing* objects is off-limits.
- Check `/?acl` and include the offending policy/ACL snippet so the fix is obvious (block public access / tighten policy).
- Free-but-existing names (`NoSuchBucket`) belong to bucket-takeover reports, not misconfig — route them to the right report type.

## Author

[Anishka Shukla](https://twitter.com/AnishkaShukla)

[Anubhav Singh](https://twitter.com/AnubhavSingh_)
