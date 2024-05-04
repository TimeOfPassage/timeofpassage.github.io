# #Linux部分命令

```shell
# 查询开放端口
iptables --list

# 入站规则
iptables -I INPUT -p tcp --dport 80 -j ACCEPT 
```

```shell
# 停用网卡

ifconfig xxxx down

# 删除网卡
brctl delbr xxxx
```

```shell
# 网络联通性测试,在待测试的机器上执行以下命令 xxxx：替换为非常规端口 (nc可以作为server端启动一个tcp的监听)
nc -l xxxx
```

```shell
# 获取当前服务pid
pid=(ps -ef |grep service-name | grep -v grep | awk 'NR==1 {print $2}')

grep -v grep, 反向选择，过滤grep行
NR==1, 表示第一行
print $2, 返回第二列的值，即PID所在列
```

```shell
# nohup 后台执行命令
nohup java -jar xx.jar > nohup.log 2>&1 &

最后& 解释：
让命令在后台执行，终端退出后命令仍旧执行

2>&1 解释：
将标准错误 2 重定向到标准输出&1，标准输出&1在被重定向到nohup.log文件中
0-stdin 标准输入
1-stdout 标准输出
2-stderr 标准错误输出
```

```shell
# sftp使用

1. 命令前+ 'l' 表示操作本机系统
	例如: lls, lcd, lpwd
2. 本机文件上传到远程主机
	> put A.txt  # 将本机当前目录下的A.txt上传到远程主机的当前目录
```
