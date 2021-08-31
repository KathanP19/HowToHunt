### Google Dorks to find Juicy Content

`inurl:example.com intitle:"index of"` <br>
`inurl:example.com intitle:"index of /" "*key.pem"` <br>
`inurl:example.com ext:log` <br>
`inurl:example.com intitle:"index of" ext:sql|xls|xml|json|csv` <br>
`inurl:example.com "MYSQL_ROOT_PASSWORD:" ext:env OR ext:yml -git` <br>
`inurl:example.com intitle:"index of" "config.db"` <br>
`inurl:example.com allintext:"API_SECRET*" ext:env | ext:yml` <br>
`inurl:example.com intext:admin ext:sql inurl:admin` <br>
`inurl:example.com allintext:username,password filetype:log` <br>
`site:example.com "-----BEGIN RSA PRIVATE KEY-----" inurl:id_rsa`<br>
`site:*.gov.* "responsible disclosure"`<br>

![t](https://miro.medium.com/max/550/1*N9W6DfGA6wxgKTiywV9aUA.png) <br>


[Refrence](https://blog.usejournal.com/how-recon-helped-samsung-protect-their-production-repositories-of-samsungtv-ecommerce-estores-4c51d6ec4fdd)


#### Other than Google, Try these dorks on various Search Engines such as Duck Duck Go, Bing etc.

## Reports (Hackerone)

### Resolved

- [Securing "Reset password" pages from bots](https://hackerone.com/reports/43807)
- [Private Grab Messages on Android App can be accessed and cached by Search Engines](https://hackerone.com/reports/221558)

### Informative

- [Information disclosure through search engines (password reset token)](https://hackerone.com/reports/322988)

### N/A

- [Research papers on yelp are getting indexed by google bots.](https://hackerone.com/reports/207435)


Author 
- [Keshav Malik](twitter.com/g0t_rOoT_) <br>
- [Naveen Prakaasham](twitter.com/NPrakaasham) <br>
- [@klaus](https://twitter.com/klaus_dev)
- [Fani Malik](https://twitter.com/fanimalikhack)
