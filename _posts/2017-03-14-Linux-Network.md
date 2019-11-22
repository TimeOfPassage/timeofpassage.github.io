---
layout: post
title: "Linux网络相关操作"
tagline: "Linux"
description: "Linux操作"
tags: ["Linux"]
---
> 嫦娥应悔偷灵药，碧海青天夜夜心。  
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
--李商隐

![风景秀丽](http://sc.jb51.net/uploads/allimg/150414/11-150414121036148.jpg)

查看本地DNS配置
> cat /etc/sysconfig/network-scripts/ifcfg-eth0

DEVICE=eth0  接口名（设备、网卡）  
HWADDR=xx:xx:xx:xx:xx:xx MAC地址   
TYPE=Ethernet 网络类型通常为Ethernet  
UUID=2e11fc24-9bbc-46b7-9e97-xxxxxxxxx  唯一标示  
ONBOOT=yes  (yes|no)系统启动的时候网络接口是否有效  
NM_CONTROLLED=yes (yes|no)是否启用NetworkManager图形界面配置工具。  
BOOTPROTO=none  [IP的配置方法(none|static|bootp|dhcp)(引导时不使用协议|静态非配IP|BOOTP协议|DHCP协议)]  
IPADDR=xxx.xxx.xxx.xxx  IP地址  
NETMASK=255.255.255.0  网络掩码   
GATEWAY=xxx.xxx.xxx.254  默认网关IP地址   
IPV6INIT=no  (yes|no)IPV6是否有效
USERCTL=no  [yes|no](非root用户是否可以控制该设备)

重新导入ifcfg-eth0网络配置文件
> /etc/init.d/network reload

网卡接口关闭与激活
> ifdown eth0 **关闭网络命令**  
> ifup eth0 **激活网络命令**

网络服务启动与关闭  

**方法一**
> service network stop  **关闭网络服务**  
> service network start **启动网络服务**  
> service network restart **重启网络服务**

**方法二**
> /etc/init.d/network stop **关闭网络服务**  
> /etc/init.d/network start **启动网络服务**  
> /etc/init.d/network restart  **重启网络服务**

网卡状态查询
> service network status

临时网卡配置
> ifconfig eth0 10.1.1.10 netmask 255.0.0.0

查看网卡接口信息，默认列出所有接口
> ifconfig

查看当前路由及网关信息
> netstat -r