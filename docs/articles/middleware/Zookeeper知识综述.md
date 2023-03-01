# Zookeeper知识综述

1. 安装过程略(使用docker比较方便，故略)

2. 进入docker执行命令

   > docker exec -it [容器id] /bin/bash
   >
   > ./zkCli.sh -server 127.0.0.1:2181

3. 列出所有节点、查看节点、创建节点、删除节点

   > ls /
   >
   > get /path
   >
   > create /path value
   >
   > delete /path

## **Java客户端使用**

0、引入pom

```xml
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.5.5</version>
</dependency>
```

1、建立连接

```java
@Slf4j
public class ZKUtil implements Watcher {

    private static String serviceAddress = "127.0.0.1:2181";
    private static CountDownLatch latch = new CountDownLatch(1);
    private static ZooKeeper zk = null;
    private static Stat stat = new Stat();
    private static final int SESSION_TIMEOUT = 30 * 1000;

    public static void main(String[] args) throws Exception {
        String path = "/test/app/vivo";
        zk = new ZooKeeper(serviceAddress, SESSION_TIMEOUT, new ZKUtil());
        latch.await();
        if (zk.exists(path, false) == null) {
            cascadeCreate(zk, path, "hello world 2");
        } else {
            zk.setData(path, "hello world 2".getBytes(), -1);
        }
        log.info(new String(zk.getData(path, true, stat)));
        System.in.read();
    }

    @Override
    public void process(WatchedEvent event) {
        /**
         *             None (-1),
         *             NodeCreated (1),
         *             NodeDeleted (2),
         *             NodeDataChanged (3),
         *             NodeChildrenChanged (4),
         *             DataWatchRemoved (5),
         *             ChildWatchRemoved (6);
         */
        log.info("{} - {} - {}", event.getType(), event.getState(), event.getPath());
        if (Event.KeeperState.SyncConnected == event.getState()) {  //zk连接成功通知事件
            if (Event.EventType.None == event.getType() && null == event.getPath()) {
                latch.countDown();
            } else if (event.getType() == Event.EventType.NodeDataChanged) {  //zk目录节点数据变化通知事件
                try {
                    System.out.println("配置已修改，新值为：" + new String(zk.getData(event.getPath(), true, stat)));
                } catch (KeeperException e) {
                    e.printStackTrace();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            } else if (event.getType() == Event.EventType.NodeDeleted) {  //zk目录节点被删除通知事件
                try {
                    System.out.println("节点已经被删除，数据置空");
                } catch (Exception e) {
                }
            }
        }
    }
}
```



2、创建永久节点

```java
zk.create(path, value.getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
```

3、级联创建永久节点

```java
private static void cascadeCreate(ZooKeeper zk, String path, String value) {
    if (StringUtils.isBlank(path)) {
        return;
    }
    try {
        if (zk.exists(path, false) != null) {
            return;
        }
        String parentPath = path.substring(0, path.lastIndexOf("/"));
        if (parentPath.length() > 0) {
            cascadeCreate(zk, parentPath, value);
            zk.create(path, value.getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
        } else {
            zk.create(path, value.getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
        }
    } catch (Exception e) {
        log.error("ZK节点创建失败:{}", e);
    }
}
```

