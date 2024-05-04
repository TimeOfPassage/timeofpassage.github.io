# ApiSix 简单使用指南

## 登陆

![image-20240119151851621](./assets/image-20240119151851621.png)

从配置文件中查找用户名和密码.

 ## 首页

![image-20240119151816343](./assets/image-20240119151816343.png)

## 第一步 创建上游服务

![image-20240119151957289](./assets/image-20240119151957289.png)

如果服务通过注册中心提供，则这里可以选择对应的服务发现类型，主要包括（Nacos、Eureka等）。ApiSix配置文件需要添加相应的发现配置。

```yaml
deployment:
  role: traditional
  role_traditional:
    config_provider: etcd
  admin:
    allow_admin:
      - 0.0.0.0/0  # Please set it to the subnet address you obtained.
                  # If not set, by default all IP access is allowed.
  etcd:
    host:
      - "http://x.x.x.x:2379"
    prefix: "/apisix"
    timeout: 30
discovery:
  nacos:
    host:
      - "http://username:password@x.x.x.x:8848"
    prefix: "/nacos/v1/"
    fetch_interval: 30    # default 30 sec
    # `weight` is the `default_weight` that will be attached to each discovered node that
    # doesn't have a weight explicitly provided in nacos results
    weight: 100           # default 100
    timeout:
      connect: 2000       # default 2000 ms
      send: 2000          # default 2000 ms
      read: 5000          # default 5000 ms
```

如果discovery没有配置，则服务调用apisix则无法找到对应的上游服务，无法进行接口调用。

## 第二步 创建路由

![image-20240119152437305](./assets/image-20240119152437305.png)

参考创建步骤，一步步设置对应的信息。

![image-20240119152547941](./assets/image-20240119152547941.png)

选择上游服务的时候，选择第一步创建的服务。然后继续下一步插件配置，可根据需要选择对应的插件。



至此，配置完成后，则可通过接口简单尝试调用了。