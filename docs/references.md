They included:

- `redirect_uri`
- `Host`
- `X-Forwarded-Host`
- File extensions
- Uploaded configuration filenames
- Multipart metadata

Each value became dangerous because a later component treated it as more trustworthy than the original source justified.

The practical testing lesson is to follow metadata after the first parser. A value may start as a harmless-looking header and later become:

- An OAuth redirect destination
- A password-reset origin
- HTML markup
- A server-side handler mapping

## 2. Validation at one layer does not guarantee safe interpretation at the next

Several labs depended on two components interpreting the same data differently.

Examples include:

- An authorization server accepting a redirect destination that the client assumed was fixed
- A proxy or middleware layer trusting `X-Forwarded-Host`
- An HTTP Host value later being inserted into HTML
- An application blocking `.php` while Apache was still allowed to execute a new extension mapping

This is why I try to map the full processing chain rather than looking only at the validation function closest to the user input.

## 3. Denylists are weak when the interpreter is flexible

The file upload lab demonstrated a classic problem with denylist thinking.

Blocking one extension answered:

> Is this filename explicitly forbidden?

It did not answer:

> Can this uploaded content ever be interpreted as executable code?

When the execution environment can be reconfigured, a denylist is especially fragile.

## 4. Security-sensitive URLs should not be built from arbitrary request metadata

The password-reset labs showed two ways this can fail:

- Trusting `Host` directly
- Trusting a forwarding header that an untrusted client can also set

A secure application should know its canonical public origin from trusted configuration or controlled infrastructure.

A random client request should not be able to decide where a password-reset secret is sent.

## 5. Browser-mediated security flows require strict binding

The OAuth lab depended on a value crossing several boundaries:

```text
victim browser
    ↓
authorization server
    ↓
attacker-controlled redirect
    ↓
legitimate client callback
```

Redirect URIs, state, authorization codes, client identity, and the initiating browser session all need appropriate binding.

A flow can use strong tokens and still fail if those tokens are delivered to or accepted in the wrong context.

## 6. A successful exploit is not the same as a root-cause explanation

For portfolio purposes, the payload is usually the least interesting part.

A stronger report explains:

- Which trust assumption failed
- Why the defence did not hold
- Which component made the unsafe decision
- How the issue should be prevented at the design level

For example:

> “Change the Host header”

is a test step.

> “The application constructs password-reset URLs from client-controlled authority metadata”

is a finding.

That distinction is the main reason I rebuilt these notes into assessment reports rather than keeping them as step-by-step lab solutions.

## Navigation

[Previous: File upload blacklist bypass](03-file-upload/01-extension-blacklist-bypass-via-htaccess.md) | [Assessment index](../README.md)