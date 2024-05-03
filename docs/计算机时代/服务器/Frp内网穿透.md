# Frp内网穿透

> 利用frp实现内网映射

## 服务端配置
```toml
bindPort = 7000 #  (云服务器需要开通防火墙规则)
auth.token = "客户端和服务端链接token"
webServer.addr = "0.0.0.0"
webServer.port = 8080 #  (视情况开通，云服务器需要开通防火墙规则)
webServer.user = "后台登陆用户名"
webServer.password = "后台登陆密码"
```

## 客户端配置
```toml
serverAddr = "公网服务器IP"
serverPort = 7000 #  (云服务器需要开通防火墙规则)
auth.token = "客户端和服务端链接token"

[[proxies]]
name = "blogweb"
type = "tcp"
localIP = "127.0.0.1"
localPort = 3000 # 客户端服务端口
remotePort = 3000 # 服务端服务端口 (云服务器需要开通防火墙规则)
```

## 验证

> 访问: [serverAddr]:[remotePort]