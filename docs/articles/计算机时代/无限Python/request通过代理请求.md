# #request通过代理请求

# 引言

通过代理该如何请求？

### 实现

```py
import requests


url = ""
payload = {}
res = requests.post(url=url, data=payload, proxies={
    "http": "socks5://username:password@ip:port",
    "https": "socks5://username:password@ip:port"
}, verify=False)
```

在requests请求的时候设置proxies信息
