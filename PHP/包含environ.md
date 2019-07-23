# 包含environ

#### 参考

- https://chybeta.github.io/2017/10/08/php%E6%96%87%E4%BB%B6%E5%8C%85%E5%90%AB%E6%BC%8F%E6%B4%9E/

#### 条件

- php以cgi方式运行，这样environ才会保持UA头。
- environ文件存储位置已知，且environ文件可读。
- 每个用户都有属于自己的environ环境

/proc/self/environ中会保存user-agent头。如果在user-agent中插入php代码，则php代码会被写入到environ中。之后再包含它，即可。
