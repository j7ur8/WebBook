### 批量下载的脚本
```py
import requests

for i in range(37):
	version='v5.1.'+str(i)+'.zip'
	url='https://github.com/top-think/think/archive/'+version
	res=requests.get(url)
	with open('download/think_'+version,'wb') as f:
		f.write(res.content)
```

### 批量创建目录
```python
import os

for i in range(38):
	os.makedirs('ThinkPHP5.1/'+'tp51'+str(i)+'/thinkphp')
```

### 移动文件

```python
def filemove(oldfile, newfile):
    weblist = os.listdir(oldfile)
    for filename in weblist:
    	shutil.move(oldfile+filename,newfile+filename)
    os.rmdir(oldfile)
```


### 解压文件
```python
def dezip(oldfile, path):
	zFile = zipfile.ZipFile(oldfile, "r")
	for fileM in zFile.namelist():
		zFile.extract(fileM, path)
	zFile.close();
```

### 文件追加写
```python
with open(vhosts,'a') as f:
	f.write(content)
```