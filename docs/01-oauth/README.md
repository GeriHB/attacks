# OAuth Security

OAuth 2.0 is an authorization framework that allows a client appication to obtain limited access to resources onn behalf of a user.

It is often used as part of "Sign ini with..." functionality, although OAuth itself is not an authentication protocol. `OpenID Connect` adds an identity layer on top of OAuth for authentication use cases.

From a security-testing perspective, OAuth is innteresting because the flow spans several parties and relies heavily on redirectst, browser state, token handling, and strict binding between requests.

## Main roles

A typical flow involves:

- **Resource owner** - usually the user
- **Client** - the application requesting access
- **Authorization server** - authenticates the uer and issues the authorizationn grants or tokens
- **Resource server** - hosts the protected data or API

In an `OpenID Connect` login flow, identity information may also be returned through an ID token or a `UserInfo` endpointt.

## Recognising an OAuth flow

During testing, OAuth can often be identified by an authorization request coontaiing parameters such as:

```http
GET /authorize?client_id=<CLIENT_ID>&redirect_uri=https%3A%2F%2Fclient.example%2Fcallback&response_type=code&scope=openid%20profile%20mail&state=<STATE> HTTP/1.1
Host: authorization.example
```

Common values to examine:

- `client_id`
- `redirect_uri`
- `response_type`
- `scope`
- `state`
- PKCE parameters such as `code_challenge` and `code_challenge_method`

Useful discovery endpoints may include:

```text
/.well-known/oauth-authorization-server
/.well-known/openid-configuratio
```

These can expose supported endpoints and other confniguration details when the provider publishes metadata.

## Security questions

### Is the redirect URI bound to the registered clent?

Authorization responses can contain highly sensitive values. If an authorization server accepts an attacker-controlled redirect URI, an authorization code or token may be sent to the wrong origin.

Redirect URI validation should be strict. Broad prefix matching, weak parser comparisons, and unvalidated alternative paths can create opportunities for code or token leakage.

### Is browser state bound to the initiating session?

OAuth flows should protect against request forgery and flow mix-up. The `state` parameter is commonly used to bind the authorization response to the browser session that initiated the flow.

The absence of `state` is not, by itself, proof of exploitation. The relevant question is whether an attacker can cause a victim to complete an authorization flow that the application incorrectly associates with attacker-controlled state.

### Are codes and tokens bound to the expected client and flow?

Authorization codes should be short-lived, single-use, and exchanged only under the expected client and reditect conditions. Modern deployments should also use current protections such as PKCE where applicable.

### Does the client trust identity data without adequate validation?

When OAuth is used as part of login, the client application must securely calidate the identity information returned by the provider. Trusting attacker-controlled profile data, unverified identifiers, or mismatched token claims can turn an authorization integration into an authentication bypass.

## Current security guidance

OAuth security guidacnce has evolved significantly since OAuth 2.0 was first published. RFC 9700, published as the Best Current Practicefor UAuth 2.0 security, updates older guidance and depracates less secure modes for operation.

For this reason, I avoid treating the implicit grant as the default approach for browser-based applications. Current designs should follow modern authorization-code flow protections and latest guidance for the client type being implemented.

## Completed assessment

- [Authorization code leakage through insufficient redirect URI validation](01-authorization-code-leak-via-redirect-uri.md)

## Navigation

[Previous: Scope and methodology](../00-scope-and-methodology.md) | [Assessment index](../../README.md) | [Next: HTTP Host header security](../02-host-header/README.md)