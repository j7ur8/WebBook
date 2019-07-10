## 基础命令

### Windows批处理中cat命令的替代者type
linux中可以使用`cat filename`的方式在console中输出文件内容，windows下可以使用`type`命令

### 清屏
```bash
cls
```

### 计算文件md5值
```bash
certutil -hashfile filename MD5
certutil -hashfile filename SHA1
certutil -hashfile filename SHA256
```

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

### Git

**git拉去远程分支到本地**
```bash
git init 
git remote add origin git@github.com:xxx/xxx.git
git fetch origin 分支
git checkout -b 本地分支 origin/远程分支
git pull origin 远程分支
```