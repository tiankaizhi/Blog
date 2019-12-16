下载 ```jdk-8u221-linux-i586.tar.gz```将其上传至 /opt 目录下进行解压

```bash
[root@localhost opt]# ll jdk-8u221-linux-i586.tar.gz
-rw-r--r--. 1 root root 198217140 8月   2 11:49 jdk-8u221-linux-i586.tar.gz
[root@localhost opt]# tar zxf jdk-8u221-linux-i586.tar.gz
# 解压之后当前 /opt 目录下生成一个名为 jdk1.8.0_221 的文件夹
[root@localhost opt]# cd jdk1.8.0_221/
[root@localhost jdk1.8.0_181]# pwd
/opt/jdk1.8.0_181
# 这就是当前 JDK8 的安装目录
```

然后配置 JDK 的环境变量。修改 ```/etc/profile``` 文件并向其中添加如下配置:

```bash
export JAVA_HOME=/opt/jdk1.8.0_221
export JRE_HOME=$JAVA_HOME/jre
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=./://$JAVA_HOME/lib:$JRE_HOME/lib
```

再执行``` source /etc/profile ```命令使配置生效，最后可以通过 java –version 命令验证 JDK 是否已经安装配置成功。如果安装配置成功，则会正确显示出 JDK 的版本信息，参考如下：

```bash
[root@localhost ~]# java -version
java version "1.8.0_221"
Java(TM) SE Runtime Environment (build 1.8.0_221-b11)
Java HotSpot(TM) Client VM (build 25.221-b11, mixed mode)
```
