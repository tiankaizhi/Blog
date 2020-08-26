## 系统信息

### 查看内核版本:

```bash
[root@webmail software-package]# cat /proc/version
Linux version 3.10.0-693.el7.x86_64 (builder@kbuilder.dev.centos.org) (gcc version 4.8.5 20150623 (Red Hat 4.8.5-16) (GCC) ) #1 SMP Tue Aug 22 21:09:27 UTC 2017
```
```bash
[root@TKZ /]# uname -a
Linux TKZ 3.10.0-1062.el7.x86_64 #1 SMP Wed Aug 7 18:08:02 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
```

```bash
[root@TKZ /]# uname -r
3.10.0-1062.el7.x86_64
```

## 防火墙

### 查看防火墙状态

```bash
[root@webmail /]# systemctl status firewalld.service
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; disabled; vendor preset: enabled)
   Active: inactive (dead)
     Docs: man:firewalld(1)
```

Active：running 表示防火墙开启；inactive(dead) 表示防火墙关闭

### 开启防火墙
```bash
[root@TKZ /]# systemctl start firewalld.service
```

### 关闭防火墙
```bash
[root@TKZ /]# systemctl stop firewalld.service
```

### 防火墙随系统开启自启动
```bash
[root@TKZ /]# systemctl enable firewalld.service
```

### 禁用防火墙系统开机自启动
```bash
[root@TKZ /]# systemctl disable firewalld.service
```

## Vim/vi 编辑器

### 未修改，退出
```bash
ESC :q
```

### 不保存
```bash
ESC :q!
```

### 保存，不退出
```bash
ESC :w
```

### 保存，退出
```bash
ESC :wq
```

### 保存，强制退出
```bash
ESC :wq！
```
