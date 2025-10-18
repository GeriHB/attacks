# OAuth

A lot of sites let you log in using *social media accounts*, and there are chances that this is built using the **OAuth 2.0 framework**.

## What is OAuth?

Enables web apps to request limited access to a user's account on another app - without exposing their login credentials.

*Usual working stages:*
- The client app requests access to some of the user's data, specifying which grant type they want to use and what kind of access.
- User is prompted to log in to the OAuth service and give their consent.
- Client app receives a unique access token that proves the permission from the user to access the requested data. (How this happens depends on the grant type).
- Client app uses this token to make API calls and fetch the relevant data from the resource server.

## OAuth authentication

The flow remains quite the same, with the difference on how the lcient app uses the data taht it receives. 

- User chooses the option to log in with the social media, and the client app uses the social media's OAuth service to request access to some data that can be used to identify the user. An example could be the email address.
- After the token is received, the client app requests this data from the resource server, typically from a dedicated `/userinfo` endpoint.
- When the data arrives, the client app uses it in place of a username to log the user - the access token from the authorization server often is used instead of a traditional password.

## Vulnerabilities and Exploitation

### Identify

To identify OAuth a straightforward way is to use Burp and check the HTTP messages when using the login option.

- First request of the flow is always to the `/authorization` endpoint containing a number of query parameters, such as:
    - `client_id`
    - `redirect_uri`
    - `response_type`

*example:* 
```http
GET /authorization?client_id=12345&redirect_uri=https://client-app.com/callback&response_type=token&scope=openid%20profile&state=ae13d489bd00e3c24 HTTP/1.1
Host: oauth-authorization-server.com
```

### Recon

If an *external* OAuth service is used, the provider can be identified from the *hostname* to which the authorization request is sent.

They often provide a *public API* and documentation that can show useful information, such as the endpoints and configurations.

Once the hostname of the authorization is known, send these `GET` requests to these standard endpoints, which can return a JSON configuration file.

```http
/.well-known/oauth-authorization-server
/.well-known/openid-configuration
```
### Exploitation 
Vulnerabilities can be:
- In the client app
    - Improper implementatin of the implicit grant type.
    - Flawed CSRF protection
- In the OAuth service
    - Leaking authorization codes and access tokens
    - Flawed scope validation
    - Unverified user registration

### Vulns in the client app

**Improper implementation of the implicit grant type**:

Implicit grant ttype is mostly recommended for *single-page* apps, but often is used in classic client-server web apps, because its simple..

1. Access token is sent from the OAuth service to the client app as URL fragment.
2. Client app processes it using JS. *Problem* is that if the app wants to maintain the session after the user closes the page, it needs to store the data (use ID and access token) somewhere.
3. Often it submits this to the server in a `POST` request and assings the user a session cookie, logging them in. But, the server doesn't have  any secrets or passwords to compare with the data submitted, so it's implicitly trusted.
4. This `POST` is exposed to the attackers, and can lead to a serious vuln if the client app doesn't properly check that the access token matches the other data in the request. Attacker simply changesthe parameters sent to the server to impersonate any user.

**Flawed CSRF protection**

`state` parameteris one of the strongly recommended component of OAuth.

It should contain an unguessable value like a hash of something tied to the user's session when it first initiates the OAuth flow.

This value is passed back and forth as a form of CSRF token.

If you see that the authorization request doesn't send a `state` parameter, i t can mean that they can initiate an OAuth flow themselves before tricking a user's browser into completing it.

Basically it means that the attacker can potentially hijack a victim user's account by binding it to their own social media account.

**Leaked authorization codes and access tokens**

The configuration of the OAuth service enables attackers to steal authorization codes or access tokens.

Depending on the grant type, either a code or token is sent via the victim's browser to the `/callback` endpoint specified in the `redirect_uri` parameter.

If the OAuth service fails to validate this URI properly, an attacker can construct a CSRF-like attack, tricking the victim's browser into initiating an OAuth flow and send the code or token to an attacker-controller `redirect_uri`.

After stealing this, the attackers can send this code to the client app legitiimate `/callback` endpoint to get access to the user's account. 

The attacker doesn't even need to know the lcient secret or the resulting access token. As long as the victim has a valid session with the OAuth service, the client appwill simply complete the code/token exchange on the attacker's behalf before logging them into the victim's account.


