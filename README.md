# Web Application Security Labs

This repository contains structured web appication security assessments performed in deliberately vulnerable training environments.

The work started as practical notes from `PortSwigger Web Security Academy labs`. I rebuild those notesi nto focused reports that document the security question, testing approach, vulnerable behavior, evidence, impact, root cause, and remediation. 
The aim is to show how I approac

## What this project covers

The current assessments focus on three areas:

- OAuth and authorization flow security.
- HTTP Host heaeder trust and password reset poisonning
- File upload validation and server-side execution

The completed lab work includes:

- Authorization code leakage through insufficient `redirect_uri` validation
- Basic passwowrd reset poisoning through attacker-controlled host information
- Password reset poisoning through `X-Forwarded-Host`
- Password reset poisoning through dangling markup
- File upload blacklist bypass through an uploaded Apache `.htaccess` file

## Repository structure

### 00 — Scope and methodology

[Testing scope, evidence handling, methodology, and limitations](docs/00-scope-and-methodology.md)

### 01 — OAuth security

[OAuth security overview](docs/01-oauth/README.md)

- [Authorization code leakage through insufficient redirect URI validation](docs/01-oauth/01-authorization-code-leak-via-redirect-uri.md)

### 02 — HTTP Host header security

[Host header trust and attack surface](docs/02-host-header/README.md)

- [Testing methodology for Host header vulnerabilities](docs/02-host-header/testing-methodology.md)
- [Password reset poisoning overview](docs/02-host-header/password-reset-poisoning/README.md)
- [Basic Host header poisoning](docs/02-host-header/password-reset-poisoning/01-basic-host-header-poisoning.md)
- [Poisoning through X-Forwarded-Host](docs/02-host-header/password-reset-poisoning/02-poisoning-via-x-forwarded-host.md)
- [Poisoning through dangling markup](docs/02-host-header/password-reset-poisoning/03-poisoning-via-dangling-markup.md)

### 03 — File upload security

[File upload attack surface](docs/03-file-upload/README.md)

- [Extension blacklist bypass through Apache .htaccess](docs/03-file-upload/01-extension-blacklist-bypass-via-htaccess.md)

### 04 — Cross-cutting lessons

[Security lessons that appear across the different vulnerability classes](docs/04-cross-cutting-lessons.md)

## Tools and techniques

The assessments primarily used:

- Burp Suite Proxy
- Burp Suite Repeater
- Browser-based workflow analysis
- HTTP request and response inspection
- Parameter and header manipulation
- Controlled attacker infrastructure provided by the training environment

## Scope and ethics

All testing documented here was performed against deliberately vulnerable training environments intended for security education.

The techniques are documented for professional development, defensive understanding, and authorised security testing. They must not be used against systems without explicit permission.

See [DISCLAIMER.md](DISCLAIMER.md) for the complete usage statement.

## Attribution

The current lab scenarios were provided by PortSwigger Web Security Academy. The reports in this repository are my own rewritten notes, analysis, HTTP examples, and security conclusions.

See [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md) for additional attribution.