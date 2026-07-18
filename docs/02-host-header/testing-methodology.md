# Testing for Host Header Vulnerabilities

Basically, we need to identify if we can modify the **Host header** and reach the target application with our request. 

If this is the case, tthen we can use the header to probe the app and observe the effects on the response.

## Supply an arbitrary Host header

Test what happens when we supply an arbitrary domain name via the Host header.

**Burp** maintains the separation between the **Host header** and the target **IP address** which allows us to supply any arbitrary Host header, and make sure that the request is sent to the intended target.

Usually there will be two cases:
- Even with an unexpected Host header we can access the target website - this can be because servevrs sometimes have a default or fallback option in case of a domain name they don't recognize. *Next, begin studying what the app does with the Host header and if it's exploitable*.
- We get `Invalid Host header` which is especially likely if the target is accessed via a CDN.

If it's the second case, we can:

## Check for flawed validation

Some websites validate if the Host header matches the **SNI** from the TLS handshake - try to understand how the website parses the Host header..

For example, some parsing algorithms omit the port from the Host header, so only the domain name is validated.

In this case, leave the domain name untouched and reach the target application, and **inject a payload via the port**.

```http
GET /example HTTP/1.1
Host: vulnerable-website.com:malicious-payload
```

Other sites apply matching logic to allow arbitrary subdomains - in this case we may be able to bypass the validation by registering an arbitrary domain name that ends with the same sequence of characters as a whitelisted one.

Or use a less-secure subdomain that we have already compromised.

## Send ambiguous requests

The code that *validates the host* and the one that does something vulnerable with it often are in different app components or even servers.

If we identify the discrepancies in how they retrieve it, we may be able to issue a request that appears to have a *different host* depending which system is looking at it.

### Inject duplicate Host headers

Often the request is blocked. But, different technologies handle this case differently - but is common that one of the two headers have precedence over the other one, thus overriding its value.

When systems disagree which header is the correct one - discrepancy that we may be able to exploit.

For example:
- Frontend gives precedence to the first header.
- Backend to the last header.

```http
GET /example HTTP/1.1
Host: website.com
Host: malicious-payload
```

So we used the first header to go to the backend and the second one to exploit it.

### Supply an absolute URL

Even that the request line specifies a relative path on the requested domain, some servers are also configured to understand requests for absolute URLs.

This can lead to discrepancies between systems, as the request line should be given precedence when routing the request.

```http
GET https://vuln_site.com/ HTTP/1.1
Host: malicious-payload
```

### Add line wrapping

A space character can uncover important behavior on how the servers will interpret the host header.

Some interpret the indented header as a wrapped line - treat it as part of the preceding header's value...

Others ignore it. So, this may uncover discrepancies.

```http
GET /example HTTP/1.1
    Host: malicious-payload
Host: website.com
```

In this case, the website may block multiple Host headers, but if the front-end *ignores* the intended header, the request will be processed as an ordinary request.

If back-end ignores the leading space and givevs the precedence to the first header in case of dupliccates, might allow us to pass values via the *wrapped* Host header.

## Inject host override headers

There are some HTTP headers that are designed to override the Host header value.

Since sites are often accessed via intermediary systems, such as *load balancers* or *reverse proxies* the Host header that the back-end receives can contain the name of such systems.

This is not relevant for them, so the frontend may inject `X-Forwarded-Host` header, which has the original value of the Host header.

So, when this is present, many frameworks use this, and this may be used sometimes to inject some malicious payload.

```http
GET /example HTTP/1.1
Host: website.com
X-Forwarded-Host: malicious-payload
```

There are also some other headers that have a similar purpose:

```http
X-Host
X-Forwarded-Server
X-HTTP-Host-Override
Forwarded
```


