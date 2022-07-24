## OAuth 2.0 Hunting Methodology
In OAuth there are 2 types of flows/grant types:
- Authorization code flow
- Implicit flow

Note: *if the oauth service uses authorization code flow then there is little to no chance of finding a bug but if the oauth service uses implicit flow then there is a good chance of finding bugs*

## How to differentiate between implicit and authorization code grant type

### <ins>Authorization code grant type</ins>

**- Authorization request**
- When you send an authorization request to the oauth service in the client application , The client application sends a request to the OAuth service's `/authorization` endpoint asking for permission to access specific user data.

>note: the endpoint name can be different according to the application like `/auth` etc. but you can identify them based on the parameters used.

the request in authorization code flow looks like:

```
GET /authorization?client_id=12345&redirect_uri=https://client-app.com/callback&response_type=code&scope=openid%20profile&state=ae13d489bd00e3c24 HTTP/1.1 
Host: oauth-authorization-server.com
```

so, in authorization code grant type the `response_type` parameter should be `code` . this code is used to request access token from the oauth service.

now, after the user login to their account with the OAuth provider and gives consent to access their data. the user will be redirected to the `/callback` endpoint that was specified in the `redirect_uri` parameter of the authorization request. The resulting `GET` request will contain the authorization code as a query parameter.

**- Authorization code grant**

```
GET /callback?code=a1b2c3d4e5f6g7h8&state=ae13d489bd00e3c24 HTTP/1.1 
Host: client-app.com
```

Rest of the stuff like access token grant and API calls are done in the back-end so you cannot see them in your proxy.

```md
**factors that determine authorization code flow:**
- Initial authorization request has `response_type=code`
- the `/callback` request contains authorization code as a parameter.
```

### <ins>Implicit grant type</ins>

**- Authorization request**
- The implicit flow starts in pretty much the same way as the authorization code flow. The only major difference is that the `response_type` parameter must be set to `token`.

```
GET /authorization?client_id=12345&redirect_uri=https://client-app.com/callback&response_type=token&scope=openid%20profile&state=ae13d489bd00e3c24 HTTP/1.1 
Host: oauth-authorization-server.com
```

**- Access Token grant**

If the user logs in and gives their consent to the request access , the oauth service redirects the user to the `/callback` endpoint but instead of sending a parameter containing an authorization code, it will send the access token and other token-specific data as a URL fragment.

```
GET /callback#access_token=z0y9x8w7v6u5&token_type=Bearer&expires_in=5000&scope=openid%20profile&state=ae13d489bd00e3c24 HTTP/1.1 
Host: client-app.com
```


```md
**factors that determine Implicit flow:**
- Initial authorization request has `response_type=token`
- the `/callback` request contains access token as a parameter.
```

---

*Now that you have determined which grant type the OAuth service uses , you can proceed to find bugs.*

### Method-1 (Auth bypass in OAuth implicit flow)
- To log the user in every time with oauth , the client application sends a POST request to the server containing user info (email-id, username) and access token to generate a session cookie.
	- so, find a POST req in http history which contains user-info and access token.
- Usually in implicit flow , the server doesn't validate the access token so you can change the parameters like email-id and/or username to impersonate another user and bypass authentication.

### Method-2 (Forced profile linking)
- This is similar to a traditional CSRF attack so the impact may not be that much.
- In this when you sign in with social media profile, you will be redirected to the social media website and then you log in with social media credentials.
- Now the next time when you log in , you will be logged in instantly. capture this request with burp.
- In the http history there would be a request similar to `/auth?client_id[...]` . In that request the redirect_uri sends the authorization code to something like `/oauth-linking`. Check if the `state` parameter is present. if its not present then it is vulnerable to CSRF attacks. because that means there is no way for server to verify if this information is from the same user.
- So absence of `state` parameter in this request is itself a vulnerability.
- Past this you can try sending the exploit link to the victim and complete the oauth flow by attaching your social media profile to their account.
	- For this copy URL of the request in burp and drop the request so that the code isn't used.
	- Turn off intercept and log out of website.
	- Now you can send this link to the victim or you can set it as an iframe on your website `<iframe src="request URL"></iframe>`.  and deliver your website link to the victim.
	- When their browser loads the `iframe`, it will complete the OAuth flow using your social media profile, attaching it to the victim account.

### Method-3 (Account hijacking via redirect_uri)
- Complete the oauth sign in flow and log out then log back in and you will be logged in instantly this time.
- Find the most recent Authorization request in http history, it should be similar to `GET /auth?client_id=[...]`.
- Check the redirect_uri param and try to change it. If you can redirect it to an external site then good , if not then try different endpoints on the same website and check if they work.
- if there is an open redirect then change the redirect_uri to your webhook site link and follow the redirect. 
- Now check for a log entry in webhook.site containing an authorization code.
- So now you can send the request url to the victim (or make an iframe as mentioned above) with redirect_uri set as your webhook site and leak their authorization codes.
- If the victim clicks on the link then you would see the authorization code in your webhook.site logs.
- now you can use this stolen code in the callback request and the rest of the OAuth flow will be completed automatically and you will be logged in as the admin user.


- [Pyr0sec](https://twitter.com/Pyr0sec)
