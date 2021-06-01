参考:
	- https://www.python.org/dev/peps/pep-0498/
	- https://www.leavesongs.com/PENETRATION/python-string-format-vulnerability.html#f


Python3.6
```python
python -c "f'''{__import__('os').system('id')}'''" 
```

**Python并没有提供一个方法，将普通字符串转换成f字符串。**
**但在eval，ssti模板注入中可用考虑**