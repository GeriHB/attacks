# SSRF via flawed request parsing

## HTTP Host header

Is a mandatory request header as of `HTTP/1.1`.

Accessing `https://portswigger.net/web-security` the browser composes the reqeust as follows:

```http
GET /web-security HTTP/1.1
Host: portswigger.net
```

Sometimes, for example when the request has been forwarded by an intermediary system, the `Host` can be changed before it reaches the back-end.

Historically, each IP address had host content for a single domain, but due to the cloud-based solution and outsorcing, now multiple websites and apps have the same IP address.

This is a common result of the following scenarios:

### Virtual Hosting

A single web server hosts multiple websites or apps. 

These distinct websites will have a different domain name, but they share a single server known as `virtual hosts`.

### Routing traffic via an intermediary

Websites are hosted on distinct back-end servers, but the traffic is routed through an intermediary system.

This can be a simple `load balancer` or a `reverse proxy server`. This is prevalent especially where clients access the website via a `CDN`.

So, all of the domain names resolve to a single IP address of the intermediary component. 

But same as with `virtual hosting` the reverse proxy or load balancer needs to know the appropriate back-end to which it should route the requests.

### How is this problem solved?

On both cases, the **Host Header** is relied on to specify the recipient. 

When a browser sends the request, the URL resolves to the IP address of the server. When the servver receivevs the request, it refers to the **Host header** to determine the intended back-end and forward the request.

## HTTP Host header attack?

If the server implicitly trusts the **Host Header**, and fails to alidate it properly, an attacker can inject harmful payloads to manipulate server-side behavior - **Host header injection**.

*Off-the-shelf* web apps usually don't know the domain they are deployed, and when they need to know the domain, for example to generate an absolute URL included in an email, they may retrieve the domain from thet Host header:

```html
<a href="https://_SERVER['HOST']/support">Contact support</a>
```

If the value is not properly escaped or validated, as it is user controllable, and used between a number of interactions also between different systems in the infrastructure, it can lead to a range of vulnerabilities:
- Web cache poisoning
- Business logic flaws in specified functionality
- Routing-based SSRF
- Classic server-side vulnerabilities, such as SQL injection.

