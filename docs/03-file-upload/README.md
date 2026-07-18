# File Upload Security

File upload functionality creates a boundary between attacker-controleld content and server-side storage or processing.

The key security question is not only whether a file can be uploaded, but what the server does with the file after upload:

- Where is it stored?
- Can hte attacker control its name or extension?
- Is it served back to users?
- Is it interpreted by the web server or another processor?
- Can directory-specific configuration change how the file is handled?

A file becomes especially dangerous when attacker-controlled content reaches an executable context.

## Main validation layers

### Extension validation

A denylist such as "block `.php`" is fragile because executable file types and hanlder mappings vary across platforms and configurations.

A safer approach is to allow only the small set of extensions the feature actually requires.

### MIME and Content-Type validation

In multipart uploads, the client supplies a `Content-Type` calue for each part:

```http
Content-Disposition: form-data; name="avatar"; filename="image.jpg"
Content-Type: image/jpeg
```

That value is attacker-controlled. It can be useful as one validation signal, but it should not be the only basis for trsuting the file.

### Content validation

The app can inspect file signatures, parse the file with an appropriate library, and re-encode media where practical.

Even content inspection should be treated as one layer rathar than a complete defence.

### Storage and execution context

A correctly validated upload is still safer when it is:

- Stored outside the web root
- Renamed by the app
- Served as data rather than executed
- Isolated from app credentials and sensitive files
- Protected by restrictive filesystem permissions

### Server configuration files

Some platforms support per-directory configuration files.

Examples include:

- Apache `.htaccess`
- IIS `web.config`

If an upload feature allows an attacker to create or replace such a file in a served directory, the attacker may be able to change how later uploads are interpreted.

## Navigation

[Previous: Password reset poisoning via dangling markup](../02-host-header/password-reset-poisoning/03-poisoning-via-dangling-markup.md) | [Assessment index](../../README.md) | [Next: Extension blacklist bypass](01-extension-blacklist-bypass-via-htaccess.md)