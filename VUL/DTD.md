参考：
- https://www.runoob.com/dtd/dtd-elements.html

XML文档均有一下5个简单的构建模块构成：
元素、属性、实体、PCDATA(会被解析器解析的文本)、CDATA(不会被解析器解析的文本)

DTD是声明XML结构的语言。对元素、属性、实体有不同的声明语法。学习XXE重点关注对实体的声明。

## 语法

元素的声明语法：
- https://www.runoob.com/dtd/dtd-elements.html
`<!ELEMENT`开头

属性的声明语法：
- https://www.runoob.com/dtd/dtd-attributes.html
`<!ATTLIST`开头

### 实体的声明语法

实体是用于定义引用普通文本或特殊字符的快捷方式的变量。
- 实体引用是对实体的引用
- 实体可在内部或外部进行声明

**内部实体**
`<!ENTITY entity-name "entity-value">`

example.xml
```xml
<?xml version="1.0" encoding="utf-8"?> 
<!DOCTYPE a [
<!ENTITY writer "Donald Duck.">
<!ENTITY copyright "Copyright runoob.com">
]>

<author>&writer;&copyright;</author>
```

**外部实体**
```xml
<?xml version="1.0" encoding="utf-8"?> 
<!DOCTYPE a [
<!ENTITY xxe SYSTEM "file:///etc/passwd" >]>
<root>
<name>&xxe;</name>
</root>
```

外部实体协议和几大语言的支持如下：
![](/images/DTD外部实体协议.png)