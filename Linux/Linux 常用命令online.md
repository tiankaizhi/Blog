
##### 修改虚拟机的 ip 地址

cd /etc/sysconfig/network-scripts/

vim ifcfg-eth0

/etc/init.d/network restart

---

##### 系统信息

一、查看Linux内核版本命令（ 2 种方法）：

cat /proc/version（**注意，这个查看内核版本信息**）
```
[root@webmail software-package]# cat /proc/version
Linux version 3.10.0-693.el7.x86_64 (builder@kbuilder.dev.centos.org) (gcc version 4.8.5 20150623 (Red Hat 4.8.5-16) (GCC) ) #1 SMP Tue Aug 22 21:09:27 UTC 2017
```
uanme -a
```
[root@webmail software-package]# uname -a
Linux webmail.unicloud.com 3.10.0-693.el7.x86_64 #1 SMP Tue Aug 22 21:09:27 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
```
uname -r (仅仅查看内核版本号)
```
[root@webmail software-package]# uname -r
3.10.0-693.el7.x86_64
```
二、查看Linux系统版本的命令（ 3 种方法）：

lsb_release -a，即可列出所有版本信息：

```
[root@webmail software-package]# lsb_release -a
-bash: lsb_release: command not found

```
这个命令适用于所有的Linux发行版，包括RedHat、SUSE、Debian…等发行版。
应该出现的结果（可能是我们公司的机器缺少东西，所以改命令使用不了）
```
[root@S-CentOS ~]# lsb_release -a
LSB Version: :base-4.0-amd64:base-4.0-noarch:core-4.0-amd64:core-4.0-noarch:graphics-4.0-amd64:graphics-4.0-noarch:printing-4.0-amd64:printing-4.0-noarch
Distributor ID: CentOS
Description: CentOS release 6.5 (Final)
Release: 6.5
Codename: Final
```

cat /etc/redhat-release，这种方法只适合 Redhat 系的 Linux（**注意，这个查看系统版本信息**）：

```
[root@webmail software-package]# cat /etc/redhat-release
CentOS Linux release 7.4.1708 (Core)
```
cat /etc/os-release ，这种方法只适合 CentOS 的 Linux（**注意，这个查看系统版本信息**）：
```
[root@webmail software-package]# cat /etc/os-release
NAME="CentOS Linux"
VERSION="7 (Core)"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="7"
PRETTY_NAME="CentOS Linux 7 (Core)"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:centos:centos:7"
HOME_URL="https://www.centos.org/"
BUG_REPORT_URL="https://bugs.centos.org/"

CENTOS_MANTISBT_PROJECT="CentOS-7"
CENTOS_MANTISBT_PROJECT_VERSION="7"
REDHAT_SUPPORT_PRODUCT="centos"
REDHAT_SUPPORT_PRODUCT_VERSION="7"
```

cat /etc/issue，此命令也适用于所有的 Linux 发行版(在我们公司机器上也不行)。
```
[root@webmail software-package]# cat /etc/issue
\S
Kernel \r on an \m


```
---

cat /proc/cpuinfo 显示CPU info的信息

date 显示系统日期

##### 关闭系统

shutdown -h now

##### 重启系统

shutdown -r now 重启

reboot 重启

##### 文件和目录

mkdir dir1 创建一个叫做 dir1 的目录

mkdir dir1 dir2 同时创建两个目录

mkdir -p /tkz/dir1/dir2 创建一个目录树

rm -f file1 删除一个叫做 file1 的文件

rmdir dir1 删除一个叫做 dir1 的目录

rm -rf dir1 删除一个叫做 dir1 的目录并同时删除其内容

rm -rf dir1 dir2 同时删除两个目录及它们的内容

mv dir1 new_dir 重命名/移动 一个目录

cp file1 file2 复制一个文件

cp dir/* . 复制一个目录下的所有文件到当前工作目录

cp -r tkz* /home/  复制 tkz 目录下所有的东西(包括文件和文件夹)到 home 目录下

##### 文件搜索

查找根目录下名称以 -i586.tar.gz

```
[root@192 /]# find / -name *-i586.tar.gz
/usr/local/apps/app-package/jdk-7u80-linux-i586.tar.gz
```

查找 local 目录下的的名称以 -i586.tar.gz 结尾的文件

```
[root@192 usr]# find local -name *-i586.tar.gz
local/apps/app-package/jdk-7u80-linux-i586.tar.gz
```
##### 打包和压缩文件

tar -cvfz zookeeper.tar.gz dir1 创建一个 gzip 格式的压缩包

tar -xvfz zookeeper.tar.gz 解压一个 gzip 格式的压缩包

tar -zxf jdk-8u211-linux-x64.tar.gz -C ../java/ 把该目录的文件解压到别的文件夹下

```
[root@webmail software-package]# ll
total 1666160
-rw-r--r-- 1 root root 194990602 Jul  8 15:38 jdk-8u211-linux-x64.tar.gz
-rw-r--r-- 1 root root 676188160 Apr 13 20:24 mysql-5.7.26-linux-glibc2.12-x86_64.tar
-rw-r--r-- 1 root root 607068160 Jun 25 14:20 mysql-8.0.16-2.el7.x86_64.rpm-bundle.tar
drwxr-xr-x 2 root root      4096 Jun 25 14:59 mysql-8.0.16-bundle-package
-rw-r--r-- 1 root root 225351680 Jun 25 11:39 otp_src_21.0.tar.gz
drwxrwxr-x 6 root root       334 Jun 25 14:11 redis-5.0.5
-rw-r--r-- 1 root root   1975750 Jun 25 14:10 redis-5.0.5.tar.gz
-rw-r--r-- 1 root root    559804 Jun 25 14:03 wget-1.14-15.el7.x86_64.rpm
[root@webmail software-package]# tar -zxf jdk-8u211-linux-x64.tar.gz -C ../java/
[root@webmail software-package]# ll

```




##### 磁盘空间

df -h 显示已经挂载的分区列表

du -sh 估算当前所在的目录的大小

du -sh dir1 估算目录 'dir1' 已经使用的磁盘空间

```
[root@192 apps]# ll
总用量 8
drwxr-xr-x. 3 root root 4096 11月 25 13:13 app
drwxr-xr-x. 2 root root 4096 11月 25 13:14 app-package
[root@192 apps]# du -sh
451M    .
[root@192 apps]# du -sh app
303M    app
[root@192 apps]# du -sh app-package
148M    app-package
```

##### 文件的权限
chmod 777 filename (r=4，w=2，x=1)

##### top 命令详细解析
https://www.jianshu.com/p/7aeb0b38f154

---
https://blog.csdn.net/shuaigexiaobo/article/details/79875730
#### yum
yum -y install wget

### 防火墙（基于 CentOS Linux release 7.4.1708 (Core)）
查看防火墙状态：systemctl status firewalld.service

```
[root@webmail /]# systemctl status firewalld.service
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; disabled; vendor preset: enabled)
   Active: inactive (dead)
     Docs: man:firewalld(1)
```
Active：running 表示防火墙开启；inactive(dead) 表示防火墙关闭

开启防火墙：systemctl start firewalld.service

关闭防火墙：systemctl stop firewalld.service

防火墙随系统开启自启动：systemctl enable firewalld.service

禁用防火墙系统开机自启动：systemctl disable firewalld.service

### SELinux
临时关闭 SELinux ：setenforce 0

临时打开 SELinux：setenforce 1

查看 SELinux 状态：getenforce

开机关闭 SELinux

vim /etc/sysconfig/selinux 文件，将 SELINUX 的值设置为 disabled（打开是：enforcing）。下次开机 SELinux 就不会启动了。

注意，此时也不能通过 setenforce 1 命令临时打开。

[root@localhost ~]# setenforce 1

setenforce: SELinux is disabled

需要修改配置文件，然后重启 linux 后，才可以再打开 SELinux

### Vim/vi 编辑器
ESC

:q   什么都没改（就是打开看一下）再退出

:q!  不保存文件（发现改错了，不需要保存），强制退出vi命令

:w   保存文件，不退出vi命令

:wq  保存文件，退出vi命令

:wq！  保存文件，退出vi命令（加!标识表示强制退出）
