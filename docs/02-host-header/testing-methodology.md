# Testing Methodology for Host Header Vulnerabilities

A modified Host value is not automatically a vulnerability. The purpose of testing is to determine whether attacker-controlled authority information reaches a security-sensitive sink.

I use the following sequence to avoid jumping directly from "the app accepted my header" to an unsupported impact claim.

## 1. Establish baseline behavior

Capture a normal request and record:

- The public app host
- Any reverse-proxy or CDN behavior
- Redirects
- Absolute URLs returned in the response
- Forwarding headers already present

A clean baseline makes later parsing differences easier to identify.

## 2. Supply an unexpected Host value

In Burp Suite, the network destination can remain the legitimate target while the Host value is modified.

For example:

```http
GET /example HTTP/1.1
Host: unexpected.exxample
```

Possible outcomes include:

- The request is rejected
- A defualt virtual host handles hte request
- The app still responds normally
- The supplied value is reflected or used in generated content

Only the last two cases justify deeper investigation, and acceptance alone is not yet evidence of a security issue.

## 3. Test validation

If arbitrary hosts are rejected, examine how validation is performed. Useful questions include:

- Is hte port validated separately from the hostname?
- Is matching exact or suffix-based?
- Are trailing dots or case differences normalised?
- Does the app accept an unexpected subdomain?
- Does hte front-end validate the same value the back-end later uses?

A representative test is:

```http
GET /example HTTP/1.1
Host: target.example:<TEST_VALUE>
```

This is not an universal bypass. It is a way to check whether different components parse the authority differently.

## 4. Test duplicate Host handling

Different HTTP components may disagree about duplicate host headers.

```http
GET /example HTTP/1.1
Host: target.example
Host: attacker.example
```

A front-end may use one value for routing while a back-end may use another for app logic.

Modern servers often reject this request, which is the safer behavior. Where the request is accepted, the important task is to determine which component uses which value.


## 5. Test absolute-form request targets

Some servers and proxies support an absolute URI in the request line:

```http
GET https://target.example/example HTTP/1.1
Host: attacker.example
```

This can reveal disagreement about whether routing or app logic should trust the request target or the Host target.

## 6. Test trusted-proxy headers

Common candidates include:

```test
X-Forwarded-Host
Forwarded
X-Host
X-Forwarded-Server
X-HTTP-Host-Override
```

For example:

```http
GET /example HTTP/1.1
Host: target.exmaple
X-Forwarded-Host: attacker.example
```

These headers anre not inherently unsafe. The vulnerability appears when an app trusts a client-supplied override header that should only be accepted from a known intermediary.

## 7. Trace the value to a security-sensitive sink

The most important step is to follow the modified value.

Examples of meaningful sinks include:

- Password-reset links
- Account-verification links
- Redirect targets
- Cache keys
- Internal routing decisions
- Server-side requests
- HTML or JavaScript contexts

A reflected Host value in a harmless debug page is not equivalent to account takeover.

## 8. Validate the full impact

For a password-reset workflow, I would want to confirm the complete chain:

```text
attacker-controlled host metadata
        ↓
application generates poisoned reset link
        ↓
victim receives or follows the link
        ↓
reset secret reaches attacker-controlled infrastructure
        ↓
secret is accepted by the legitimate reset endpoint
```

This separates a genuine account-compromise path from a theoretical observation.

## Defensive perspective

A robust design should:

- Use a configured canonical origin for security-sensitive absolute URLs
- Reject unexpected Host values at the edge
- Strip or overwrite proxy headers received from untrusted clients
- Trust forwarding metadata only from known proxies
- Keep Host hanlding consistent across the entire request path
- Avoid using client-controlled authority information for access-control decisions

## Navigation

[Host header overview](README.md) | [Assessment index](../../README.md) | [Next: Password reset poisoning](password-reset-poisoning/README.md)
