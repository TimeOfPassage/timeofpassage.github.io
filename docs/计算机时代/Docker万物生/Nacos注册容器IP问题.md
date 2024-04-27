# Nacos注册容器IP问题

> 当使用docker部署应用服务的时候，而应用服务又需要注册到nacos使用dubbo功能，则默认情况下注册的容器的IP，
> 这里提供一种解决方案。

## 1. 通过`DUBBO_IP_TO_REGISTRY`解决

1. `DUBBO_IP_TO_REGISTRY`: 设置需要注册的服务IP（一般是机器IP）
2. `DUBBO_PORT_TO_REGISTRY`: 设置需要注册的服务Dubbo端口

```bash
# 示例
docker run --name xxx -e DUBBO_IP_TO_REGISTRY=192.168.16.10 -e DUBBO_PORT_TO_REGISTRY=20880 --net=service -p 20880:20880 xxx:latest
```


## Reference

* https://github.com/apache/dubbo/issues/13579