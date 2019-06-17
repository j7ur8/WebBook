参考：
- https://mp.weixin.qq.com/s?__biz=MjM5MDkwNjA2Nw==&mid=2650374608&idx=1&sn=9dc025a8bbc372c0819a7f99f253ad1b&chksm=beb0826c89c70b7ab18819390d9d21bc1ce21ba6a0ab3bc80198d53f01d73802732a967c4289&xtrack=1&scene=0&subscene=131&clicktime=1557228517&ascene=7&devicetype=android-26&version=2700043a&nettype=3gnet&abtest_cookie=BAABAAoACwASABMABgBWmR4AyZkeANyZHgDimR4A6JkeAPaZHgAAAA%3D%3D&lang=zh_CN&pass_ticket=rEQf3kgA20VvAwczDhyDV1wRbeeg%2B9oj39QMFxAabbiBVKUBboNLwupzMrtAPvYA&wx_header=1

IIS 6.0在处理含有特殊符号的文件路径或者文件名会出现逻辑错误，从而造成文件解析漏洞。  

1. 目录解析
```
/xx.asp/xx.jpg
```
文件夹的名字为`.asp`的文件夹，其目录内的任何扩展名的文件都被IIS当作asp文件来解析并执行。默认的可执行文件还有`cer`、`asa`、`cdx`。  

2. 文件解析
`wooyun.asp;.jpg`此文件也算时asp可执行文件