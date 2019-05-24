参考：
https://www.w3schools.com/xml/xml_whatis.asp

xml的语法和HTML差不多，但是XML是用来传输数据的。其语句可以受到DTD等声明语言的控制和限制。

## XML语法：
1. 声明：	
	`<?xml version="1.0" encoding="utf-8"?>`
2. 标签
* 属性值:要加引号，不加引号就是错的
* 必须正确嵌套，<></>
* 大小写敏感
* 实体引用
	* `<`在xml元素中会代表元素开始，需要用实体`&lt;`替换，有五个预定义实体![](/images/XML_预定义实体.png)

只有`<`和`&`是非法的，大于号等是合法的

3. 注释
- `<!-- asdf -->`

4. 空格
空格会被保留，但是连续空格会被合并为1个

5. 换行
在 Windows 应用程序中，换行通常以一对字符来存储：回车符（CR）和换行符（LF）。
在 Unix 和 Mac OSX 中，使用 LF 来存储新行。
在旧的 Mac 系统中，使用 CR 来存储新行。
XML 以 LF 存储换行。

6. 命名空间
使用前缀时，用于前缀的命名空间必须被定义。在xmlns中定义。
```xml
<h:table xmlns:h="http://www.w3.org/TR/html4/">
<h:tr>
<h:td>Apples</h:td>
<h:td>Bananas</h:td>
</h:tr>
</h:table>

<f:table xmlns:f="http://www.w3cschool.cc/furniture">
<f:name>African Coffee Table</f:name>
<f:width>80</f:width>
<f:length>120</f:length>
</f:table>
```

元素和属性的区别：
- https://www.runoob.com/dtd/dtd-el_vs-attr.html