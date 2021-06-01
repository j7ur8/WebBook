参考：
	- https://xz.aliyun.com/t/2908#toc-3
## Tornado
渲染函数：
- render
通过模板文件名加载模板，然后更新模板引擎中的命名空间，添加一些全局函数或其他对象，然后生成并返回渲染好的 html内容  
- render_string
依次调用render_string及相关渲染函数生成的内容，最后调用 finish 直接输出给客户端。  


handler.application可用访问整个Tornado  
poc
```python
{{handler.application.settings}}
{{handler.settings}}
```