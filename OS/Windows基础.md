## 基础命令

### Windows批处理中cat命令的替代者type
linux中可以使用`cat filename`的方式在console中输出文件内容，windows下可以使用`type`命令

### 清屏
`cls`


## 配置环境

### sublime

**PHP编译环境**   
php7012.sublime-build  
```json
{
	"cmd": ["E:\\SecurityTools\\phpStudy\\PHPTutorial\\php\\php-7.0.12-nts\\php.exe", "$file"],
	"file_regex": "php$",
	"selector": "source.php",
	"encoding": "gbk"
}
```

**python编译环境**
```json
{
    "cmd": ["E:\\Languages\\Python27\\python.exe","-u","$file"],
    "file_regex":"^[ ]*File \"(...*?)\", line ([0-9]*)",
    "selector":"source.python",
    "encoding": "gbk"
}
```