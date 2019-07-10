### 绕过mime_content_type()函数
参考：
- https://www.php.net/manual/zh/function.mime-content-type.php
- https://bugs.php.net/bug.php?id=75280
- https://xz.aliyun.com/t/4029

一番搜索有2中方法可以绕过。    
**首先是php的issue，关于`<?[空格]`可被检测为`text/plain`的问题。**  
**其次是minme_content_type检测的是文件头，可以制造一个合法的图片头＋恶意文件内容的图片**
```python
with open('1.jpg','wb') as f:
	f.write(b'\x89PNG\r\n\x1a\n')
print(b'\x89PNG')
```
采用以上方法来制作一个合法图片头的头片。  
以下为几种图片类型的最短可过检测的字符串。   
- png->`\x89PNG\r\n\x1a\n`，如果取在`\n`前面，那么mine_content_type()函数将检测为`application/octet-stream`
- jpg->`\xFF\xD8`
- gif->`\x47\x49\x46\x38`
- svg直接text写以下内容即可。
```xml
<?xml version="1.0" standalone="no"?>
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 20010904//EN"
 "http://www.w3.org/TR/2001/REC-SVG-20010904/DTD/svg10.dtd">
<svg version="1.0" xmlns="http://www.w3.org/2000/svg">
</svg>
```