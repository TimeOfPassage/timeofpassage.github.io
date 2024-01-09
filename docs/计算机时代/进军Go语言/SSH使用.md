# #SSH使用

# 引言

> 本文主要描述了如何用Go实现SSH相关链接及操作

# 假设信息

## 机器A(通过密码认证)

> IP: 10.5.0.120
>
> Port: 22
>
> Username: aDev
>
> Password: aaaaaa

## 机器B(通过证书认证)

> IP: 10.5.0.121
>
> Port: 22
>
> Username: bDev
>
> CertPath: /home/path/b.pem

## 机器C(先登陆机器A， 在通过A开隧道进行证书认证)

> IP: 10.5.0.122
>
> Port: 22
>
> Username: cDev
>
> CertPath: /home/path/c.pem

# 简单实现

新建项目：略

## 引入依赖库

```bash

go get golang.org/x/crypto/ssh
```

## 构建链接配置结构体

```go
type SSHConnectConfig struct {
	Host     string
	Port     int
	Username string
	// AuthType: 0-password 1-ssh pem; default value is 0
	AuthType       int    `json:"auth_type"`
	Password       string `json:"password"`
	PemKeyPath     string
	PemKeyPassword string
}
```

## 基础链接方法

	根据机器信息构建sshConnectConfig对象， 然后通过`NewSshClient`​方法构建Client。一旦获取到Client， 则可以通过Client进行一些基本操作。 

```go
func buildAuthMethod(authType int, password string, pemKeyPath string) ([]ssh.AuthMethod, ConnectionError) {
	// 判断是密码登录还是密钥登录
	if authType == 0 {
		return []ssh.AuthMethod{ssh.Password(password)}, nil
	}
	// 加载pem
	pemBytes, err := os.ReadFile(pemKeyPath)
	if err != nil {
		log.Fatalf("read file from path error: %v", err)
		return nil, err
	}
	key, err := ssh.ParsePrivateKey(pemBytes)
	if err != nil {
		log.Fatalf("byte parse private key error: %v", err)
		return nil, err
	}
	return []ssh.AuthMethod{ssh.PublicKeys(key)}, nil
}

func NewSshClient(connConfig SSHConnectConfig) (*ssh.Client, ConnectionError) {
	log.Printf("connecting %s ", connConfig.Host)
	authMethods, err := buildAuthMethod(connConfig.AuthType, connConfig.Password, connConfig.PemKeyPath)
	if err != nil {
		return nil, err
	}
	// 构建SSH客户端配置
	config := &ssh.ClientConfig{
		User:            connConfig.Username,
		Auth:            authMethods,
		HostKeyCallback: ssh.InsecureIgnoreHostKey(),
	}
	address := net.JoinHostPort(connConfig.Host, strconv.Itoa(connConfig.Port))
	// 开始链接
	client, err := ssh.Dial("tcp", address, config)
	if err != nil {
		log.Fatalf("unable to connect to server, %v", err)
		return nil, err
	}
	log.Printf("%s connected.", connConfig.Host)
	return client, nil
}
```

	通过封装上述方法， 我们可以轻易实现机器A和机器B的链接。 但是如何链接假设信息里的机器C呢。很明显， 上述方法不行， 我们来增强下链接方法

## 通过跳板机链接

	通过传入顺序数组，来进行链接。如果数组元素只有一个，则直接创建返回Client对象；如果数组存在多个，则通过在Client新建隧道来链接下一跳机器返回下一跳机器的Client对象。

```go
func jumpToNext(client *ssh.Client, connConfig SSHConnectConfig) (*ssh.Client, ConnectionError) {
	nextAddress := net.JoinHostPort(connConfig.Host, strconv.Itoa(connConfig.Port))
	nextConn, err := client.Dial("tcp", nextAddress)
	if err != nil {
		log.Fatalf("Failed create conn from client, %v", err)
		return nil, err
	}
	authMethods, err := buildAuthMethod(connConfig.AuthType, connConfig.Password, connConfig.PemKeyPath)
	if err != nil {
		return nil, err
	}
	config := &ssh.ClientConfig{
		User:            connConfig.Username,
		Auth:            authMethods,
		HostKeyCallback: ssh.InsecureIgnoreHostKey(),
	}
	ncc, channels, reqs, err := ssh.NewClientConn(nextConn, nextAddress, config)
	if err != nil {
		return nil, err
	}
	nextClient := ssh.NewClient(ncc, channels, reqs)
	return nextClient, nil
}

func NewSshClientWithProxy(configs []SSHConnectConfig) (*ssh.Client, ConnectionError) {
	if len(configs) == 0 {
		return nil, errors.New("链接信息不能为空")
	}
	client, err := NewSshClient(configs[0])
	if len(configs) == 1 {
		return client, err
	}
	var proxyClient = client
	for i := 1; i < len(configs); i++ {
		client, err = jumpToNext(proxyClient, configs[i])
		if err != nil {
			return nil, err
		}
		proxyClient = client
	}
	return client, err
}
```

至此，我们支持实现了机器A、机器B、机器C的连接。

# 需求

	知识是通过使用思源笔记的docker镜像构建的，通过前置Nginx将请求收敛到80端口下，通过腾讯云免费SSL证书+Nginx实现https安全链接。但是由于使用轻量云服务器，有可能被攻击的风险。所以需要实现一个小工具：**执行后可以自动链接云服务器实现笔记数据备份至当前电脑目录的自动化工具。**

## 要求

* 用Go语言实现
* 支持手动执行 ~定时执行可暂不实现~
* 支持Mac、Windows平台
* 终端程序，不需要有界面

‍
