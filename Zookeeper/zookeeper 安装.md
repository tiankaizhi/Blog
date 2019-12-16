## ZooKeeper 单机安装步骤

### 下载安装包，解压缩。

下载```zookeeper-3.4.6.tar.gz```安装包，将其复制到```/opt```目录下，然后解压缩：

```shell
[root@localhost app-package]# ll zookeeper-3.4.6.tar.gz
-rw-r--r--. 1 root root 17699306 8月   1 15:09 zookeeper-3.4.6.tar.gz
[root@localhost opt]# tar zxf zookeeper-3.4.6.tar.gz
# 解压之后当前 /opt 目录下生成一个名为 zookeeper-3.4.6 的文件夹
[root@localhost opt]# cd zookeeper-3.4.6
[root@localhost zookeeper-3.4.12]# pwd
/opt/zookeeper-3.4.6
```

### 配置环境变量

向```/etc/profile```配置文件中添加如下内容，并执行```source /etc/profile```命令使配置生效：

```shell
export ZOOKEEPER_HOME=/opt/zookeeper-3.4.6
export PATH=$PATH:$ZOOKEEPER_HOME/bin
```
### 修改 Zookeeper 配置文件

修改 ZooKeeper 的配置文件。首先进入```$ZOOKEEPER_HOME/conf```目录，并将```zoo_sample.cfg```文件修改为```zoo.cfg```

```shell
[root@localhost zookeeper-3.4.6]# cd conf
[root@localhost conf]# mv zoo_sample.cfg zoo.cfg
```

然后修改```zoo.cfg```配置文件，```zoo.cfg```文件的内容参考如下：

```shell
# Zookeeper 服务器之间或客户端与服务器之间维持心跳的时间间隔，也就是每个 tickTime 时间就会发送一个心跳，单位为 ms.
tickTime=2000
# 投票选举新 leader 的初始化时间
# 此配置表示，允许 follower 连接并同步到 leader 的初始化连接时间，它以 tickTime 的倍数来表示，默认为10。
# 当超过设置倍数的 tickTime 时间，则连接失败。如果在设定的时间段内，半数以上的跟随者未能完成同步，领导者
# 便会宣布放弃领导地位，进行另一次的领导选举。如果 zk 集群环境数量确实很大，同步数据的时间会变长，因此
# 这种情况下可以适当调大该参数。
initLimit=10
# leader 与 follower 心跳检测最大容忍时间，也就是每次发送心跳的最大响应时间超过 syncLimit * tickTime，leader 认为
# follower “死掉”，从服务器列表中删除 follower，所有关联到这个 follower 的客户端将连接到另外一个 follower。
syncLimit=5
# 数据目录
dataDir=/tmp/zookeeper/data
# 日志目录
dataLogDir=/tmp/zookeeper/log
# ZooKeeper 对外服务端口
clientPort=2181
```

默认情况下，Linux 系统中没有```/tmp/zookeeper/data```和```/tmp/zookeeper/log```这两个目录，所以接下来还要创建这两个目录：

```shell
[root@localhost conf]# mkdir -p /tmp/zookeeper/data
[root@localhost conf]# mkdir -p /tmp/zookeeper/log
```

### 设置 Zookeeper 服务编号

在```${dataDir}```目录（也就是```/tmp/zookeeper/data```）下创建一个```myid```文件，并写入一个数值，比如 0。```myid``` 文件里存放的是服务器的编号。

### 启动 Zookeeper 服务

启动 Zookeeper 服务，详情如下：

```shell
[root@localhost conf]# zkServer.sh start
JMX enabled by default
Using config: /opt/zookeeper-3.4.6/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
```

可以通过```zkServer.sh status```命令查看 Zookeeper 服务状态，示例如下：

```shell
[root@localhost ]# zkServer.sh status
JMX enabled by default
Using config: /opt/zookeeper-3.4.6/bin/../conf/zoo.cfg
Mode: Standalone
```

## ZooKeeper 集群安装

以上是关于 ZooKeeper 单机模式的安装与配置，一般在生产环境中使用的都是集群模式，集群模式的配置也比较简单，相比单机模式而言只需要修改一些配置即可。下面以 3 台机器为例来配置一个 ZooKeeper 集群。首先在这 3 台机器的```/etc/hosts```文件中添加 3 台集群的 IP 地址与机器域名的映射，示例如下（3 个 IP 地址分别对应 3 台机器;不配置域名和 IP 地址映射直接使用 IP 地址也可以）：

```properties
192.168.0.2 node1
192.168.0.3 node2
192.168.0.4 node3
```

然后在这 3 台机器的```zoo.cfg```文件中添加以下配置：

```properties
server.0=node1:2888:3888
server.1=node2:2888:3888
server.2=node3:2888:3888
```

为了便于讲解上面的配置，这里抽象出一个公式，即```server.A=B:C:D```。其中 A 是一个数字，代表服务器的编号，就是前面所说的 myid 文件里面的值。集群中每台服务器的编号都必须唯一，所以要保证每台服务器中的 myid 文件中的值不同。B 代表服务器的 IP 地址。C 表示服务器与集群中的 leader 服务器交换信息的端口。D 表示选举时服务器相互通信的端口。如此，集群模式的配置就告一段落，可以在这 3 台机器上各自执行```zkServer.sh start```命令来启动服务。
