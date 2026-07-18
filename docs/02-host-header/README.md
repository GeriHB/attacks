# HTTP Host Header Security

The HTTP Host value tells a server which authority the client intends to reach. 

In HTTP/1.1, the `Host` hedaer is required. 

In HTTP/1, equivalent authority information is carried by the `:authority` pseudo-header, although security tools often continue to present hte concept in familiar Host-header terms.

Modern web apps frequently sit behind reverse proxies, load balancers, CDNs, API gateways, and other intermediaries. As a result, several components may parse or rewrite host information before the request reaches app code.

That becomes a security issue when attacker-controlled host metadata is trusted for something sensitive.

## Where the Host valiue is used

Apps and supporting infrastructure may use host information for:

- Virtual-host routing
- Selecting a back-end service
- Constructing absolute URLs
- Generating links in emails
- Cache keys
- Redirects
- Access-control decisions
- Internal routing

The header is supplient by the client, so it should not automatically be treated as a trusted soruce of hte apps canonical public origin.

## Common failure patterns

### Building absolute URLs from untrusted request metadata

A password-reset workflow may generate a link like:

```text
https://<HOST_FROM_REQUEST>/forgot-password?token<RESET_TOKEN>
```

If the app trusts an attackerc-controlled host value, the secret token can be sent to an attacker-controlled domain.

### Trusting proxy override headers from untrusted clients

Headers such as `X-Forwarded-Host` may be added by trusted infrastructure, but an app can become vulnerable if arbitrary internet clietns are also allowed to set them and the app gives those values precedence.

### Inconsistent parsing between components

A front end and a back end may disagree about:

- Which Host value is authoritative
- Whether a port is part of validation
- How duplicate Host headers are handled
- Whether an absolute-form request target overrides the Host header
- Which forwarding header should be trusted

A value rejected or normalised by one component may still reach another component in a different form.

## Potential consequences

Unsafe Host handling can contribute to:

- Password reset poisoning
- Web cache poisoning
- Routing-based SSRF
- Authentication or access-control mistakes
- Open redirects
- Injection into generated content

The exact impact depends on where the host value is used.

## Completed work

- [Testing methodology for Host header vulnerabilities](testing-methodology.md)
- [Password reset poisoning overview](password-reset-poisoning/README.md)
- [Basic Host header poisoning](password-reset-poisoning/01-basic-host-header-poisoning.md)
- [Poisoning through X-Forwarded-Host](password-reset-poisoning/02-poisoning-via-x-forwarded-host.md)
- [Poisoning through dangling markup](password-reset-poisoning/03-poisoning-via-dangling-markup.md)

## Navigation

[Previous: OAuth security](../01-oauth/README.md) | [Assessment index](../../README.md) | [Next: Host header testing methodology](testing-methodology.md)