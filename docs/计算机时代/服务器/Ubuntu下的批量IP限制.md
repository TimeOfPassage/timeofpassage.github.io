# 特定批量IP限制访问

## 场景

服务器部署网站信息需要限制部分国家不允许访问。

## 方案

1. 方案一：通过apache、nginx这些进行限制。
2. 方案二：通过iptables防火墙规则实现。

本文采用方案二实现。

## 下载国家IP信息

地址一：https://www.ip2location.com/free/visitor-blocker
地址二：http://www.ipdeny.com/ipblocks

本文选用地址二，并用中国示例

etc. http://www.ipdeny.com/ipblocks/data/countries/cn.zone

## 安装ipset工具

```bash
apt-get install ipset -y
```

## 创建ipset的规则

```bash
#ipset -N cnip hash:net
```

上面命令创建了一个名为: cnip 的规则

## 将cn.zone中的ip段信息添加到cnip

```bash
for i in $(cat /cn.zone);
do
ipset -A cnip $i;
done
```

上述cn.zone地址信息请用户参考自己具体路径。


## 禁止cnip中的ip访问80和443端口

```bash
iptables -I INPUT -p tcp -m set --match-set cnip src --dport 80 -j DROP
iptables -I INPUT -p tcp -m set --match-set cnip src --dport 443 -j DROP
```

# 重启后自动生效(适用于Ubuntu系列)

## 保存iptables规则

```bash
iptables-save > /etc/iptables.rules
```

## 进入/etc/network/if-pre-up.d/目录新建bash脚本

```bash
#!/bin/bash
iptables-restore < /etc/iptables.rules
```

至此，禁止ip访问设置结束...