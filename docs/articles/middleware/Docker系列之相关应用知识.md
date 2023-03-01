#  Docker相关应用操作

## 1.如何设置环境变量
```bash
-e：设置环境变量
例: docker run -e "ZOO_INIT_LIMIT=10" --name some-zookeeper --restart always -d 31z4/zookeeper
```

## 2.如何进入容器内部
```bash
docker exec -i -t [container-id] bin/bash
```

## 3.Docker应用

## Zookeeper

```bash
启动服务端：(2181,2888,3888,8080)
docker run --name [custome-zookeeper-name] -p 2181:2181 --restart always -d [image-id]
```

```bash
更新启动：
docker update -p 2181:2181 -p 2888:2888 --restart always -d [image-id]
```

```bash
启动客户端：(zkCli)
docker run -it --rm --link [custome-zookeeper-name]:zookeeper [image-id] zkCli.sh -server zookeeper
```

## Apache Dubbo Admin

```bash
启动：
docker run -p 8080:8080 -e "admin.registry.address=zookeeper://127.0.0.1:2181" -e "admin.config-center=zookeeper://127.0.0.1:2181" -e "admin.metadata-report.address=zookeeper://127.0.0.1:2181" apache/dubbo-admin
```

```
信息描述：
	version: '3'
	services:
		zookeeper:
		image: zookeeper
		ports:
			- 2181:2181
		admin:
		image: apache/dubbo-admin
		depends_on:
			- zookeeper
		ports:
			- 8080
		environment:
			- admin.registry.address=zookeeper://zookeeper:2181
			- admin.config-center=zookeeper://zookeeper:2181
			- admin.metadata-report.address=zookeeper://zookeeper:2181
```

## Redis

```bash
简单启动：	
docker run --name [some-redis] -d [redis:5.0.6]
```

```bash
启动持久化实例：
docker run --name [some-redis] -d [redis:5.0.6] redis-server --appendonly yes
```

```bash
客户端链接：
docker run -it --rm --link [Redis01]:redis [redis:5.0.6] redis-cli -h [Redis01]
```

## MongoDB

```bash
创建挂载目录:
docker volume create mongo_data_db
docker volume create mongo_data_configdb
```

```bash
启动：
docker run -d --name mongo -v mongo_data_configdb:/e/data/configdb -v mongo_data_db:/e/data/db -p 27017:27017 mongo --auth
```

```bash
初始化管理账号：
docker exec -it mongo     mongo              admin
```

```bash
创建最高权限：
db.createUser({ user: 'admin', pwd: 'admin', roles: [ { role: "root", db: "admin" } ] });
```

```bash
测试联通性：
docker run -it --rm --link mongo:mongo mongo:3.4 mongo -u admin -p admin --authenticationDatabase admin mongo/admin
```

## ElasticSearch

```bash
创建网络:
docker network create mynetwork --driver=bridge
```

```bash
启动：
docker run --name elasticsearch -d -e ES_JAVA_OPTS="-Xms512m -Xmx512m" -p 9200:9200 -p 9300:9300 --network=mynetwork elasticsearch:6.7.0
```

```bash
启动
docker run -it -d -e ELASTICSEARCH_URL=http://elasticsearch:9200 --name kibana -p 5601:5601 --network=mynetwork kibana:6.7.0
```

```bash
启动客户端(elasticsearch-head)：
docker run -d -p 9100:9100 docker.io/mobz/elasticsearch-head:5 --name elasticsearch-head
```

## RabbitMQ 

```bash
启动：
docker run --name rabbitmq -d -p 15672:15672 -p 5672:5672 rabbitmq:3.7-management
```

### 如何进入RabbitMQ容器内部

```bash
添加用户：用户名为root,密码为123456
rabbitmqctl add_user root 123456
```

```bash
赋予权限：
abbitmqctl set_permissions -p / root ".*" ".*" ".*"
```

```bash
赋予adminstrator角色权限
rabbitmqctl set_user_tags root adminstrator
```

```bash
查看所有用户
rabbitmqctl list_users
```

## Kafka 

```bash
docker run  -d --name kafka -p 9092:9092 -e KAFKA_BROKER_ID=0 -e KAFKA_ZOOKEEPER_CONNECT=192.168.2.65:2181 -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://192.168.2.65:9092 -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 -t wurstmeister/kafka
```

[Kafka](https://juejin.im/entry/6844903829624848398)

### 如何进入kafka容器进行相关操作

```bash
# 测试进入kafka容器
docker exec -it kafka /bin/bash

# 进入kafka目录
cd opt/kafka/

# 启动消息发送方(producer) 
./bin/kafka-console-producer.sh --broker-list localhost:9092 --topic mykafka

# 克隆会话(打开新的cmd窗口，进入新的kafka容器)

# 进入kafka目录
cd opt/kafka/

# 启动消息接收方(consumer)
./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic mykafka --from-beginning

# 测试消息发送与消息接收
# 在消息发送方输入123456
# 在消息接收方查看 如果看到123456 消息发送完成
```