# Password Reset Poisoning

Password reset poisoning occurs when an app builds a password-reset link using attacker-controlled authority information.

The reset token itself may be cryptographically strong and completely unpredictable, but the failure is elsewhere:

- the app sends that valid secret to the wrong origin because it trusted request metadata when constructing the URL.

## Normal reset flow

A simplified secure flow is:

```text
user requests a password reset
        ↓
server generates a high-entropy, short-lived token
        ↓
token is associated with the intended account
        ↓
app sends a link to the user's verified email address
        ↓
user presents the token to the legitimate app
        ↓
token is invalidated after ose or expiry
```

The security of the token does not help if the link containing it points to an attacker-controlled host.

## Poisoned flow

```text
attacker requests a reset for the victim
        ↓
attacker influences Host or trusted-proxy metadata
        ↓
app generates a reset URL using attacker-controlled authority
        ↓
victim follows hte link
        ↓
reset secret is disclosed to attacker infrastructure
        ↓
attacker replays the secret against the legitimate app
```

## What the three labs demonstrate

The completed labs show three related but distinct trust failures:

1. **Direct Host poisoning** - the app uses hte request Host directly
2. **Proxy-header poisoning** - the app keeps hte legitimate Host but trusts `X-Forwarded-Host` supplied by the client
3. **Dangling markup** - attacker-controlled Host data reaches an HTML context and causes later email content to be exfiltrated

The common lesson is that security-sensitive content is being generated from request metadata that the attacker can influecnce.

## Completed assessments

- [Basic Host header poisoning](01-basic-host-header-poisoning.md)
- [Poisoning through X-Forwarded-Host](02-poisoning-via-x-forwarded-host.md)
- [Poisoning through dangling markup](03-poisoning-via-dangling-markup.md)

## Navigation

[Previous: Host header testing methodology](../testing-methodology.md) | [Assessment index](../../../README.md) | [Next: Basic Host header poisoning](01-basic-host-header-poisoning.md)
