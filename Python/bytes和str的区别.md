## bytes和str的区别
```python
a='中文'
a_b=a.encode('utf-8')
print(a,type(a),len(a))			# 中文 <class 'str'> 2
print(a_b,type(a_b),len(a_b))	# b'\xe4\xb8\xad\xe6\x96\x87' <class 'bytes'> 6

print('\n')
a='aaa'
a_b=a.encode('utf-8')
print(a,type(a),len(a))			# aaa <class 'str'> 3
print(a_b,type(a_b),len(a_b))	# b'aaa' <class 'bytes'> 3

print('\n')
a='aaa中国'
a_b=a.encode('utf-8')
print(a,type(a),len(a))			# aaa中国 <class 'str'> 5
print(a_b,type(a_b),len(a_b))	# b'aaa\xe4\xb8\xad\xe5\x9b\xbd' <class 'bytes'> 9
```

bytes一般对二进制使用，但也可以对在ascii码范围得的字符使用。
str对文本使用。