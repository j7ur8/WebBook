参考：
- https://0x1.im/blog/php/php-function-getimagesize.html

(PHP 4, PHP 5, PHP 7)

解析：
```c
PHPAPI int php_getimagetype(php_stream * stream, char *filetype TSRMLS_DC)
{
	...
	if (!memcmp(filetype, php_sig_gif, 3)) {
		return IMAGE_FILETYPE_GIF;
	} else if (!memcmp(filetype, php_sig_jpg, 3)) {
		return IMAGE_FILETYPE_JPEG;
	} else if (!memcmp(filetype, php_sig_png, 3)) {
		...
	}
}

PHPAPI const char php_sig_gif[3] = {'G', 'I', 'F'};
...
PHPAPI const char php_sig_png[8] = {(char) 0x89, (char) 0x50, (char) 0x4e, (char) 0x47,
                                    (char) 0x0d, (char) 0x0a, (char) 0x1a, (char) 0x0a};
```

可以看出来 image type 是根据文件流的前几个字节（文件头）来判断的

demo
```
<?php
print_r(getimagesize('test.php'));
```

exp.png
```
<?php phpinfo(); ?>

3C3F 7068 7020 7068 7069 6E66 6F28 293B 203F 3E

8950 4E47 0D0A 1A0A 3C3F 7068 7020 7068 7069 6E66 6F28 293B 203F 3E
```