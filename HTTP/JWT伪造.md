```python
import jwt
import base64
## pyjwt==0.4.3
public = "-----BEGIN PUBLIC KEY-----\nMIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDMRTzM9ujkHmh42aXG0aHZk/PK\nomh6laVF+c3+D+klIjXglj7+/wxnztnhyOZpYxdtk7FfpHa3Xh4Pkpd5VivwOu1h\nKk3XQYZeMHov4kW0yuS+5RpFV1Q2gm/NWGY52EaQmpCNFQbGNigZhu95R2OoMtuc\nIC+LX+9V/mpyKe9R3wIDAQAB\n-----END PUBLIC KEY-----"
print(jwt.encode({"name": "zxczxc","priv": "admin"}, key=public, algorithm='HS256'))

```