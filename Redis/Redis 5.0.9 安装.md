[TOC]

---

## 系统环境

### 系统版本
```bash
[root@TKZ opt]# cat /etc/redhat-release
CentOS Linux release 7.7.1908 (Core)
```

### 内核版本
```bash
[root@TKZ opt]# uname -a
Linux TKZ 3.10.0-1062.el7.x86_64 #1 SMP Wed Aug 7 18:08:02 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
```

## 安装步骤

### 安装 gcc 依赖

Redis 是用 C 语言开发，安装之前先安装 gcc 环境。

执行

```bash
yum install -y gcc
```

```bash
[root@TKZ opt]# yum install -y gcc
已加载插件：fastestmirror, langpacks
Repository base is listed more than once in the configuration
Repository updates is listed more than once in the configuration
Repository extras is listed more than once in the configuration
Repository centosplus is listed more than once in the configuration
Loading mirror speeds from cached hostfile
 * base: mirrors.ustc.edu.cn
 * extras: mirrors.ustc.edu.cn
 * updates: mirrors.ustc.edu.cn
正在解决依赖关系
--> 正在检查事务
---> 软件包 gcc.x86_64.0.4.8.5-39.el7 将被 安装
--> 正在处理依赖关系 cpp = 4.8.5-39.el7，它被软件包 gcc-4.8.5-39.el7.x86_64 需要
--> 正在处理依赖关系 glibc-devel >= 2.2.90-12，它被软件包 gcc-4.8.5-39.el7.x86_64 需要
--> 正在检查事务
---> 软件包 cpp.x86_64.0.4.8.5-39.el7 将被 安装
---> 软件包 glibc-devel.x86_64.0.2.17-292.el7 将被 安装
--> 正在处理依赖关系 glibc-headers = 2.17-292.el7，它被软件包 glibc-devel-2.17-292.el7.x86_64 需要
--> 正在处理依赖关系 glibc-headers，它被软件包 glibc-devel-2.17-292.el7.x86_64 需要
--> 正在检查事务
---> 软件包 glibc-headers.x86_64.0.2.17-292.el7 将被 安装
--> 正在处理依赖关系 kernel-headers >= 2.2.1，它被软件包 glibc-headers-2.17-292.el7.x86_64 需要
--> 正在处理依赖关系 kernel-headers，它被软件包 glibc-headers-2.17-292.el7.x86_64 需要
--> 正在检查事务
---> 软件包 kernel-headers.x86_64.0.3.10.0-1062.18.1.el7 将被 安装
--> 解决依赖关系完成

依赖关系解决

====================================================================================================================================================================
 Package                                  架构                             版本                                             源                                 大小
====================================================================================================================================================================
正在安装:
 gcc                                      x86_64                           4.8.5-39.el7                                     base                               16 M
为依赖而安装:
 cpp                                      x86_64                           4.8.5-39.el7                                     base                              5.9 M
 glibc-devel                              x86_64                           2.17-292.el7                                     base                              1.1 M
 glibc-headers                            x86_64                           2.17-292.el7                                     base                              687 k
 kernel-headers                           x86_64                           3.10.0-1062.18.1.el7                             updates                           8.8 M

事务概要
====================================================================================================================================================================
安装  1 软件包 (+4 依赖软件包)

总下载量：33 M
安装大小：59 M
Downloading packages:
警告：/var/cache/yum/x86_64/7/base/packages/glibc-headers-2.17-292.el7.x86_64.rpm: 头V3 RSA/SHA256 Signature, 密钥 ID f4a80eb5: NOKEY.4 MB/s | 2.2 MB  00:00:22 ETA
glibc-headers-2.17-292.el7.x86_64.rpm 的公钥尚未安装
(1/5): glibc-headers-2.17-292.el7.x86_64.rpm                                                                                                 | 687 kB  00:00:01
(2/5): glibc-devel-2.17-292.el7.x86_64.rpm                                                                                                   | 1.1 MB  00:00:02
kernel-headers-3.10.0-1062.18.1.el7.x86_64.rpm 的公钥尚未安装       40% [=======================                                  ] 1.8 MB/s |  13 MB  00:00:10 ETA
(3/5): kernel-headers-3.10.0-1062.18.1.el7.x86_64.rpm                                                                                        | 8.8 MB  00:00:07
(4/5): cpp-4.8.5-39.el7.x86_64.rpm                                                                                                           | 5.9 MB  00:00:12
(5/5): gcc-4.8.5-39.el7.x86_64.rpm                                                                                                                 |  16 MB  00:23:26
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------
总计                                                                                                                                       24 kB/s |  33 MB  00:23:26
从 file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7 检索密钥
导入 GPG key 0xF4A80EB5:
 用户ID     : "CentOS-7 Key (CentOS 7 Official Signing Key) <security@centos.org>"
 指纹       : 6341 ab27 53d7 8a78 a7c2 7bb1 24c6 a8a7 f4a8 0eb5
 软件包     : centos-release-7-7.1908.0.el7.centos.x86_64 (@anaconda)
 来自       : /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
警告：RPM 数据库已被非 yum 程序修改。
** 发现 11 个已存在的 RPM 数据库问题， 'yum check' 输出如下：
icedtea-web-1.7.1-1.el7.x86_64 有缺少的需求 java-1.8.0-openjdk
icedtea-web-1.7.1-1.el7.x86_64 有缺少的需求 jpackage-utils
icedtea-web-1.7.1-1.el7.x86_64 有缺少的需求 jpackage-utils
jline-1.0-8.el7.noarch 有缺少的需求 java >= ('0', '1.5', None)
jline-1.0-8.el7.noarch 有缺少的需求 jpackage-utils
2:postfix-2.10.1-7.el7.x86_64 有缺少的需求 libmysqlclient.so.18()(64bit)
2:postfix-2.10.1-7.el7.x86_64 有缺少的需求 libmysqlclient.so.18(libmysqlclient_18)(64bit)
rhino-1.7R5-1.el7.noarch 有缺少的需求 jpackage-utils
rhino-1.7R5-1.el7.noarch 有缺少的需求 jpackage-utils
tagsoup-1.2.1-8.el7.noarch 有缺少的需求 jpackage-utils
tagsoup-1.2.1-8.el7.noarch 有缺少的需求 jpackage-utils >= ('0', '1.6', None)
  正在安装    : cpp-4.8.5-39.el7.x86_64                                                                                                                               1/5
  正在安装    : kernel-headers-3.10.0-1062.18.1.el7.x86_64                                                                                                            2/5
  正在安装    : glibc-headers-2.17-292.el7.x86_64                                                                                                                     3/5
  正在安装    : glibc-devel-2.17-292.el7.x86_64                                                                                                                       4/5
  正在安装    : gcc-4.8.5-39.el7.x86_64                                                                                                                               5/5
  验证中      : glibc-devel-2.17-292.el7.x86_64                                                                                                                       1/5
  验证中      : gcc-4.8.5-39.el7.x86_64                                                                                                                               2/5
  验证中      : kernel-headers-3.10.0-1062.18.1.el7.x86_64                                                                                                            3/5
  验证中      : glibc-headers-2.17-292.el7.x86_64                                                                                                                     4/5
  验证中      : cpp-4.8.5-39.el7.x86_64                                                                                                                               5/5

已安装:
  gcc.x86_64 0:4.8.5-39.el7

作为依赖被安装:
  cpp.x86_64 0:4.8.5-39.el7        glibc-devel.x86_64 0:2.17-292.el7        glibc-headers.x86_64 0:2.17-292.el7        kernel-headers.x86_64 0:3.10.0-1062.18.1.el7

完毕！
```

### 下载 Redis

Redis 官网下载速度慢的同学可以去国内镜像下载。

执行

```bash
wget https://mirrors.huaweicloud.com/redis/redis-5.0.9.tar.gz
```

```bash
[root@TKZ opt]# wget https://mirrors.huaweicloud.com/redis/redis-5.0.9.tar.gz
--2020-04-25 13:35:09--  https://mirrors.huaweicloud.com/redis/redis-5.0.9.tar.gz
正在解析主机 mirrors.huaweicloud.com (mirrors.huaweicloud.com)... 117.78.24.34, 117.78.24.32, 117.78.24.36
正在连接 mirrors.huaweicloud.com (mirrors.huaweicloud.com)|117.78.24.34|:443... 已连接。
已发出 HTTP 请求，正在等待回应... 200 OK
长度：1986574 (1.9M) [application/octet-stream]
正在保存至: “redis-5.0.9.tar.gz”

100%[================================================================================================================================>] 1,986,574   1.43MB/s 用时 1.3s

2020-04-25 13:35:11 (1.43 MB/s) - 已保存 “redis-5.0.9.tar.gz” [1986574/1986574])
```

```bash
[root@TKZ opt]# ll
-rw-r--r--.  1 root  root    1986574 4月  24 00:14 redis-5.0.9.tar.gz
```

### 解压 Redis

执行

```bash
tar -zxf redis-5.0.9.tar.gz
```

```bash
[root@TKZ opt]# tar -zxf redis-5.0.9.tar.gz
[root@TKZ opt]# ll
drwxrwxr-x.  6 root  root       4096 4月  17 18:45 redis-5.0.9
-rw-r--r--.  1 root  root    1986574 4月  24 00:14 redis-5.0.9.tar.gz
```

### 切换到 redis 解压目录下，执行编译

```bash
cd redis-5.0.9/

make
```

```bash
[root@TKZ opt]# cd redis-5.0.9/
[root@TKZ redis-5.0.9]# ll
总用量 276
-rw-rw-r--.  1 root root 119270 4月  17 18:45 00-RELEASENOTES
-rw-rw-r--.  1 root root     53 4月  17 18:45 BUGS
-rw-rw-r--.  1 root root   2381 4月  17 18:45 CONTRIBUTING
-rw-rw-r--.  1 root root   1487 4月  17 18:45 COPYING
drwxrwxr-x.  6 root root    124 4月  17 18:45 deps
-rw-rw-r--.  1 root root     11 4月  17 18:45 INSTALL
-rw-rw-r--.  1 root root    151 4月  17 18:45 Makefile
-rw-rw-r--.  1 root root   6888 4月  17 18:45 MANIFESTO
-rw-rw-r--.  1 root root  20555 4月  17 18:45 README.md
-rw-rw-r--.  1 root root  61797 4月  17 18:45 redis.conf
-rwxrwxr-x.  1 root root    275 4月  17 18:45 runtest
-rwxrwxr-x.  1 root root    280 4月  17 18:45 runtest-cluster
-rwxrwxr-x.  1 root root    373 4月  17 18:45 runtest-moduleapi
-rwxrwxr-x.  1 root root    281 4月  17 18:45 runtest-sentinel
-rw-rw-r--.  1 root root   9710 4月  17 18:45 sentinel.conf
drwxrwxr-x.  3 root root   4096 4月  17 18:45 src
drwxrwxr-x. 11 root root    182 4月  17 18:45 tests
drwxrwxr-x.  8 root root   4096 4月  17 18:45 utils
[root@TKZ redis-5.0.9]# make
cd src && make all
make[1]: 进入目录“/opt/redis-5.0.9/src”
    CC Makefile.dep
make[1]: 离开目录“/opt/redis-5.0.9/src”
make[1]: 进入目录“/opt/redis-5.0.9/src”
rm -rf redis-server redis-sentinel redis-cli redis-benchmark redis-check-rdb redis-check-aof *.o *.gcda *.gcno *.gcov redis.info lcov-html Makefile.dep dict-benchmark
(cd ../deps && make distclean)
make[2]: 进入目录“/opt/redis-5.0.9/deps”
(cd hiredis && make clean) > /dev/null || true
(cd linenoise && make clean) > /dev/null || true
(cd lua && make clean) > /dev/null || true
(cd jemalloc && [ -f Makefile ] && make distclean) > /dev/null || true
(rm -f .make-*)
make[2]: 离开目录“/opt/redis-5.0.9/deps”
(rm -f .make-*)
echo STD=-std=c99 -pedantic -DREDIS_STATIC='' >> .make-settings
echo WARN=-Wall -W -Wno-missing-field-initializers >> .make-settings
echo OPT=-O2 >> .make-settings
echo MALLOC=jemalloc >> .make-settings
echo CFLAGS= >> .make-settings
echo LDFLAGS= >> .make-settings
echo REDIS_CFLAGS= >> .make-settings
echo REDIS_LDFLAGS= >> .make-settings
echo PREV_FINAL_CFLAGS=-std=c99 -pedantic -DREDIS_STATIC='' -Wall -W -Wno-missing-field-initializers -O2 -g -ggdb   -I../deps/hiredis -I../deps/linenoise -I../deps/lua/src -DUSE_JEMALLOC -I../deps/jemalloc/include >> .make-settings
echo PREV_FINAL_LDFLAGS=  -g -ggdb -rdynamic >> .make-settings
(cd ../deps && make hiredis linenoise lua jemalloc)
make[2]: 进入目录“/opt/redis-5.0.9/deps”
(cd hiredis && make clean) > /dev/null || true
(cd linenoise && make clean) > /dev/null || true
(cd lua && make clean) > /dev/null || true
(cd jemalloc && [ -f Makefile ] && make distclean) > /dev/null || true
(rm -f .make-*)
(echo "" > .make-cflags)
(echo "" > .make-ldflags)
MAKE hiredis
cd hiredis && make static
make[3]: 进入目录“/opt/redis-5.0.9/deps/hiredis”
cc -std=c99 -pedantic -c -O3 -fPIC  -Wall -W -Wstrict-prototypes -Wwrite-strings -g -ggdb  net.c
cc -std=c99 -pedantic -c -O3 -fPIC  -Wall -W -Wstrict-prototypes -Wwrite-strings -g -ggdb  hiredis.c
cc -std=c99 -pedantic -c -O3 -fPIC  -Wall -W -Wstrict-prototypes -Wwrite-strings -g -ggdb  sds.c
cc -std=c99 -pedantic -c -O3 -fPIC  -Wall -W -Wstrict-prototypes -Wwrite-strings -g -ggdb  async.c
cc -std=c99 -pedantic -c -O3 -fPIC  -Wall -W -Wstrict-prototypes -Wwrite-strings -g -ggdb  read.c
ar rcs libhiredis.a net.o hiredis.o sds.o async.o read.o
make[3]: 离开目录“/opt/redis-5.0.9/deps/hiredis”
MAKE linenoise
cd linenoise && make
make[3]: 进入目录“/opt/redis-5.0.9/deps/linenoise”
cc  -Wall -Os -g  -c linenoise.c
make[3]: 离开目录“/opt/redis-5.0.9/deps/linenoise”
MAKE lua
cd lua/src && make all CFLAGS="-O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL -DREDIS_STATIC='' " MYLDFLAGS="" AR="ar rcu"
make[3]: 进入目录“/opt/redis-5.0.9/deps/lua/src”
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL -DREDIS_STATIC=''    -c -o lapi.o lapi.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL -DREDIS_STATIC=''    -c -o lcode.o lcode.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL -DREDIS_STATIC=''    -c -o ldebug.o ldebug.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL -DREDIS_STATIC=''    -c -o ldo.o ldo.c
ldo.c: 在函数‘f_parser’中:
ldo.c:496:7: 警告：未使用的变量‘c’ [-Wunused-variable]
   int c = luaZ_lookahead(p->z);
       ^
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL -DREDIS_STATIC=''    -c -o ldump.o ldump.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL -DREDIS_STATIC=''    -c -o lfunc.o lfunc.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL -DREDIS_STATIC=''    -c -o lgc.o lgc.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL -DREDIS_STATIC=''    -c -o llex.o llex.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL -DREDIS_STATIC=''    -c -o lmem.o lmem.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL -DREDIS_STATIC=''    -c -o lobject.o lobject.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL -DREDIS_STATIC=''    -c -o lopcodes.o lopcodes.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL -DREDIS_STATIC=''    -c -o lparser.o lparser.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL -DREDIS_STATIC=''    -c -o lstate.o lstate.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL -DREDIS_STATIC=''    -c -o lstring.o lstring.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL -DREDIS_STATIC=''    -c -o ltable.o ltable.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL -DREDIS_STATIC=''    -c -o ltm.o ltm.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL -DREDIS_STATIC=''    -c -o lundump.o lundump.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL -DREDIS_STATIC=''    -c -o lvm.o lvm.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL -DREDIS_STATIC=''    -c -o lzio.o lzio.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL -DREDIS_STATIC=''    -c -o strbuf.o strbuf.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL -DREDIS_STATIC=''    -c -o fpconv.o fpconv.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL -DREDIS_STATIC=''    -c -o lauxlib.o lauxlib.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL -DREDIS_STATIC=''    -c -o lbaselib.o lbaselib.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL -DREDIS_STATIC=''    -c -o ldblib.o ldblib.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL -DREDIS_STATIC=''    -c -o liolib.o liolib.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL -DREDIS_STATIC=''    -c -o lmathlib.o lmathlib.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL -DREDIS_STATIC=''    -c -o loslib.o loslib.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL -DREDIS_STATIC=''    -c -o ltablib.o ltablib.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL -DREDIS_STATIC=''    -c -o lstrlib.o lstrlib.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL -DREDIS_STATIC=''    -c -o loadlib.o loadlib.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL -DREDIS_STATIC=''    -c -o linit.o linit.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL -DREDIS_STATIC=''    -c -o lua_cjson.o lua_cjson.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL -DREDIS_STATIC=''    -c -o lua_struct.o lua_struct.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL -DREDIS_STATIC=''    -c -o lua_cmsgpack.o lua_cmsgpack.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL -DREDIS_STATIC=''    -c -o lua_bit.o lua_bit.c
ar rcu liblua.a lapi.o lcode.o ldebug.o ldo.o ldump.o lfunc.o lgc.o llex.o lmem.o lobject.o lopcodes.o lparser.o lstate.o lstring.o ltable.o ltm.o lundump.o lvm.o lzio.o strbuf.o fpconv.o lauxlib.o lbaselib.o ldblib.o liolib.o lmathlib.o loslib.o ltablib.o lstrlib.o loadlib.o linit.o lua_cjson.o lua_struct.o lua_cmsgpack.o lua_bit.o    # DLL needs all object files
ranlib liblua.a
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL -DREDIS_STATIC=''    -c -o lua.o lua.c
cc -o lua  lua.o liblua.a -lm
liblua.a(loslib.o)：在函数‘os_tmpname’中：
loslib.c:(.text+0x28c): 警告：the use of `tmpnam' is dangerous, better use `mkstemp'
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL -DREDIS_STATIC=''    -c -o luac.o luac.c
cc -O2 -Wall -DLUA_ANSI -DENABLE_CJSON_GLOBAL -DREDIS_STATIC=''    -c -o print.o print.c
cc -o luac  luac.o print.o liblua.a -lm
make[3]: 离开目录“/opt/redis-5.0.9/deps/lua/src”
MAKE jemalloc
cd jemalloc && ./configure --with-version=5.1.0-0-g0 --with-lg-quantum=3 --with-jemalloc-prefix=je_ --enable-cc-silence CFLAGS="-std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops " LDFLAGS=""
configure: WARNING: unrecognized options: --enable-cc-silence
checking for xsltproc... /usr/bin/xsltproc
checking for gcc... gcc
checking whether the C compiler works... yes
checking for C compiler default output file name... a.out
checking for suffix of executables...
checking whether we are cross compiling... no
checking for suffix of object files... o
checking whether we are using the GNU C compiler... yes
checking whether gcc accepts -g... yes
checking for gcc option to accept ISO C89... none needed
checking whether compiler is cray... no
checking whether compiler supports -std=gnu11... yes
checking whether compiler supports -Wall... yes
checking whether compiler supports -Wshorten-64-to-32... no
checking whether compiler supports -Wsign-compare... yes
checking whether compiler supports -Wundef... yes
checking whether compiler supports -Wno-format-zero-length... yes
checking whether compiler supports -pipe... yes
checking whether compiler supports -g3... yes
checking how to run the C preprocessor... gcc -E
checking for g++... no
checking for c++... no
checking for gpp... no
checking for aCC... no
checking for CC... no
checking for cxx... no
checking for cc++... no
checking for cl.exe... no
checking for FCC... no
checking for KCC... no
checking for RCC... no
checking for xlC_r... no
checking for xlC... no
checking whether we are using the GNU C++ compiler... no
checking whether g++ accepts -g... no
checking whether g++ supports C++14 features by default... no
checking whether g++ supports C++14 features with -std=c++14... no
checking whether g++ supports C++14 features with -std=c++0x... no
checking whether g++ supports C++14 features with +std=c++14... no
checking whether g++ supports C++14 features with -h std=c++14... no
configure: No compiler with C++14 support was found
checking for grep that handles long lines and -e... /usr/bin/grep
checking for egrep... /usr/bin/grep -E
checking for ANSI C header files... yes
checking for sys/types.h... yes
checking for sys/stat.h... yes
checking for stdlib.h... yes
checking for string.h... yes
checking for memory.h... yes
checking for strings.h... yes
checking for inttypes.h... yes
checking for stdint.h... yes
checking for unistd.h... yes
checking whether byte ordering is bigendian... no
checking size of void *... 8
checking size of int... 4
checking size of long... 8
checking size of long long... 8
checking size of intmax_t... 8
checking build system type... x86_64-pc-linux-gnu
checking host system type... x86_64-pc-linux-gnu
checking whether pause instruction is compilable... yes
checking number of significant virtual address bits... 48
checking for ar... ar
checking for nm... nm
checking for gawk... gawk
checking malloc.h usability... yes
checking malloc.h presence... yes
checking for malloc.h... yes
checking whether malloc_usable_size definition can use const argument... no
checking for library containing log... -lm
checking whether __attribute__ syntax is compilable... yes
checking whether compiler supports -fvisibility=hidden... yes
checking whether compiler supports -fvisibility=hidden... no
checking whether compiler supports -Werror... yes
checking whether compiler supports -herror_on_warning... no
checking whether tls_model attribute is compilable... yes
checking whether compiler supports -Werror... yes
checking whether compiler supports -herror_on_warning... no
checking whether alloc_size attribute is compilable... yes
checking whether compiler supports -Werror... yes
checking whether compiler supports -herror_on_warning... no
checking whether format(gnu_printf, ...) attribute is compilable... yes
checking whether compiler supports -Werror... yes
checking whether compiler supports -herror_on_warning... no
checking whether format(printf, ...) attribute is compilable... yes
checking for a BSD-compatible install... /usr/bin/install -c
checking for ranlib... ranlib
checking for ld... /usr/bin/ld
checking for autoconf... false
checking for memalign... yes
checking for valloc... yes
checking whether compiler supports -O3... yes
checking whether compiler supports -O3... no
checking whether compiler supports -funroll-loops... yes
checking configured backtracing method... N/A
checking for sbrk... yes
checking whether utrace(2) is compilable... no
checking whether a program using __builtin_unreachable is compilable... yes
checking whether a program using __builtin_ffsl is compilable... yes
checking LG_PAGE... 12
checking pthread.h usability... yes
checking pthread.h presence... yes
checking for pthread.h... yes
checking for pthread_create in -lpthread... yes
checking dlfcn.h usability... yes
checking dlfcn.h presence... yes
checking for dlfcn.h... yes
checking for dlsym... no
checking for dlsym in -ldl... yes
checking whether pthread_atfork(3) is compilable... yes
checking whether pthread_setname_np(3) is compilable... yes
checking for library containing clock_gettime... none required
checking whether clock_gettime(CLOCK_MONOTONIC_COARSE, ...) is compilable... yes
checking whether clock_gettime(CLOCK_MONOTONIC, ...) is compilable... yes
checking whether mach_absolute_time() is compilable... no
checking whether compiler supports -Werror... yes
checking whether syscall(2) is compilable... yes
checking for secure_getenv... yes
checking for sched_getcpu... yes
checking for sched_setaffinity... yes
checking for issetugid... no
checking for _malloc_thread_cleanup... no
checking for _pthread_mutex_init_calloc_cb... no
checking for TLS... yes
checking whether C11 atomics is compilable... no
checking whether GCC __atomic atomics is compilable... yes
checking whether GCC __sync atomics is compilable... yes
checking whether Darwin OSAtomic*() is compilable... no
checking whether madvise(2) is compilable... yes
checking whether madvise(..., MADV_FREE) is compilable... no
checking whether madvise(..., MADV_DONTNEED) is compilable... yes
checking whether madvise(..., MADV_DO[NT]DUMP) is compilable... yes
checking whether madvise(..., MADV_[NO]HUGEPAGE) is compilable... yes
checking whether to force 32-bit __sync_{add,sub}_and_fetch()... no
checking whether to force 64-bit __sync_{add,sub}_and_fetch()... no
checking for __builtin_clz... yes
checking whether Darwin os_unfair_lock_*() is compilable... no
checking whether Darwin OSSpin*() is compilable... no
checking whether glibc malloc hook is compilable... yes
checking whether glibc memalign hook is compilable... yes
checking whether pthreads adaptive mutexes is compilable... yes
checking whether compiler supports -D_GNU_SOURCE... yes
checking whether compiler supports -Werror... yes
checking whether compiler supports -herror_on_warning... no
checking whether strerror_r returns char with gnu source is compilable... yes
checking for stdbool.h that conforms to C99... yes
checking for _Bool... yes
configure: creating ./config.status
config.status: creating Makefile
config.status: creating jemalloc.pc
config.status: creating doc/html.xsl
config.status: creating doc/manpages.xsl
config.status: creating doc/jemalloc.xml
config.status: creating include/jemalloc/jemalloc_macros.h
config.status: creating include/jemalloc/jemalloc_protos.h
config.status: creating include/jemalloc/jemalloc_typedefs.h
config.status: creating include/jemalloc/internal/jemalloc_preamble.h
config.status: creating test/test.sh
config.status: creating test/include/test/jemalloc_test.h
config.status: creating config.stamp
config.status: creating bin/jemalloc-config
config.status: creating bin/jemalloc.sh
config.status: creating bin/jeprof
config.status: creating include/jemalloc/jemalloc_defs.h
config.status: creating include/jemalloc/internal/jemalloc_internal_defs.h
config.status: creating test/include/test/jemalloc_test_defs.h
config.status: executing include/jemalloc/internal/public_symbols.txt commands
config.status: executing include/jemalloc/internal/private_symbols.awk commands
config.status: executing include/jemalloc/internal/private_symbols_jet.awk commands
config.status: executing include/jemalloc/internal/public_namespace.h commands
config.status: executing include/jemalloc/internal/public_unnamespace.h commands
config.status: executing include/jemalloc/internal/size_classes.h commands
config.status: executing include/jemalloc/jemalloc_protos_jet.h commands
config.status: executing include/jemalloc/jemalloc_rename.h commands
config.status: executing include/jemalloc/jemalloc_mangle.h commands
config.status: executing include/jemalloc/jemalloc_mangle_jet.h commands
config.status: executing include/jemalloc/jemalloc.h commands
configure: WARNING: unrecognized options: --enable-cc-silence
===============================================================================
jemalloc version   : 5.1.0-0-g0
library revision   : 2

CONFIG             : --with-version=5.1.0-0-g0 --with-lg-quantum=3 --with-jemalloc-prefix=je_ --enable-cc-silence 'CFLAGS=-std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops ' LDFLAGS=
CC                 : gcc
CONFIGURE_CFLAGS   : -std=gnu11 -Wall -Wsign-compare -Wundef -Wno-format-zero-length -pipe -g3 -fvisibility=hidden -O3 -funroll-loops
SPECIFIED_CFLAGS   : -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops
EXTRA_CFLAGS       :
CPPFLAGS           : -D_GNU_SOURCE -D_REENTRANT
CXX                : g++
CONFIGURE_CXXFLAGS :
SPECIFIED_CXXFLAGS :
EXTRA_CXXFLAGS     :
LDFLAGS            :
EXTRA_LDFLAGS      :
DSO_LDFLAGS        : -shared -Wl,-soname,$(@F)
LIBS               : -lm  -lpthread -ldl
RPATH_EXTRA        :

XSLTPROC           : /usr/bin/xsltproc
XSLROOT            :

PREFIX             : /usr/local
BINDIR             : /usr/local/bin
DATADIR            : /usr/local/share
INCLUDEDIR         : /usr/local/include
LIBDIR             : /usr/local/lib
MANDIR             : /usr/local/share/man

srcroot            :
abs_srcroot        : /opt/redis-5.0.9/deps/jemalloc/
objroot            :
abs_objroot        : /opt/redis-5.0.9/deps/jemalloc/

JEMALLOC_PREFIX    : je_
JEMALLOC_PRIVATE_NAMESPACE
                   : je_
install_suffix     :
malloc_conf        :
autogen            : 0
debug              : 0
stats              : 1
prof               : 0
prof-libunwind     : 0
prof-libgcc        : 0
prof-gcc           : 0
fill               : 1
utrace             : 0
xmalloc            : 0
log                : 0
lazy_lock          : 0
cache-oblivious    : 1
cxx                : 0
===============================================================================
cd jemalloc && make CFLAGS="-std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops " LDFLAGS="" lib/libjemalloc.a
make[3]: 进入目录“/opt/redis-5.0.9/deps/jemalloc”
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -DJEMALLOC_NO_PRIVATE_NAMESPACE -o src/jemalloc.sym.o src/jemalloc.c
nm -a src/jemalloc.sym.o | gawk -f include/jemalloc/internal/private_symbols.awk > src/jemalloc.sym
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -DJEMALLOC_NO_PRIVATE_NAMESPACE -o src/arena.sym.o src/arena.c
nm -a src/arena.sym.o | gawk -f include/jemalloc/internal/private_symbols.awk > src/arena.sym
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -DJEMALLOC_NO_PRIVATE_NAMESPACE -o src/background_thread.sym.o src/background_thread.c
nm -a src/background_thread.sym.o | gawk -f include/jemalloc/internal/private_symbols.awk > src/background_thread.sym
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -DJEMALLOC_NO_PRIVATE_NAMESPACE -o src/base.sym.o src/base.c
nm -a src/base.sym.o | gawk -f include/jemalloc/internal/private_symbols.awk > src/base.sym
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -DJEMALLOC_NO_PRIVATE_NAMESPACE -o src/bin.sym.o src/bin.c
nm -a src/bin.sym.o | gawk -f include/jemalloc/internal/private_symbols.awk > src/bin.sym
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -DJEMALLOC_NO_PRIVATE_NAMESPACE -o src/bitmap.sym.o src/bitmap.c
nm -a src/bitmap.sym.o | gawk -f include/jemalloc/internal/private_symbols.awk > src/bitmap.sym
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -DJEMALLOC_NO_PRIVATE_NAMESPACE -o src/ckh.sym.o src/ckh.c
nm -a src/ckh.sym.o | gawk -f include/jemalloc/internal/private_symbols.awk > src/ckh.sym
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -DJEMALLOC_NO_PRIVATE_NAMESPACE -o src/ctl.sym.o src/ctl.c
nm -a src/ctl.sym.o | gawk -f include/jemalloc/internal/private_symbols.awk > src/ctl.sym
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -DJEMALLOC_NO_PRIVATE_NAMESPACE -o src/div.sym.o src/div.c
nm -a src/div.sym.o | gawk -f include/jemalloc/internal/private_symbols.awk > src/div.sym
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -DJEMALLOC_NO_PRIVATE_NAMESPACE -o src/extent.sym.o src/extent.c
nm -a src/extent.sym.o | gawk -f include/jemalloc/internal/private_symbols.awk > src/extent.sym
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -DJEMALLOC_NO_PRIVATE_NAMESPACE -o src/extent_dss.sym.o src/extent_dss.c
nm -a src/extent_dss.sym.o | gawk -f include/jemalloc/internal/private_symbols.awk > src/extent_dss.sym
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -DJEMALLOC_NO_PRIVATE_NAMESPACE -o src/extent_mmap.sym.o src/extent_mmap.c
nm -a src/extent_mmap.sym.o | gawk -f include/jemalloc/internal/private_symbols.awk > src/extent_mmap.sym
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -DJEMALLOC_NO_PRIVATE_NAMESPACE -o src/hash.sym.o src/hash.c
nm -a src/hash.sym.o | gawk -f include/jemalloc/internal/private_symbols.awk > src/hash.sym
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -DJEMALLOC_NO_PRIVATE_NAMESPACE -o src/hooks.sym.o src/hooks.c
nm -a src/hooks.sym.o | gawk -f include/jemalloc/internal/private_symbols.awk > src/hooks.sym
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -DJEMALLOC_NO_PRIVATE_NAMESPACE -o src/large.sym.o src/large.c
nm -a src/large.sym.o | gawk -f include/jemalloc/internal/private_symbols.awk > src/large.sym
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -DJEMALLOC_NO_PRIVATE_NAMESPACE -o src/log.sym.o src/log.c
nm -a src/log.sym.o | gawk -f include/jemalloc/internal/private_symbols.awk > src/log.sym
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -DJEMALLOC_NO_PRIVATE_NAMESPACE -o src/malloc_io.sym.o src/malloc_io.c
nm -a src/malloc_io.sym.o | gawk -f include/jemalloc/internal/private_symbols.awk > src/malloc_io.sym
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -DJEMALLOC_NO_PRIVATE_NAMESPACE -o src/mutex.sym.o src/mutex.c
nm -a src/mutex.sym.o | gawk -f include/jemalloc/internal/private_symbols.awk > src/mutex.sym
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -DJEMALLOC_NO_PRIVATE_NAMESPACE -o src/mutex_pool.sym.o src/mutex_pool.c
nm -a src/mutex_pool.sym.o | gawk -f include/jemalloc/internal/private_symbols.awk > src/mutex_pool.sym
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -DJEMALLOC_NO_PRIVATE_NAMESPACE -o src/nstime.sym.o src/nstime.c
nm -a src/nstime.sym.o | gawk -f include/jemalloc/internal/private_symbols.awk > src/nstime.sym
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -DJEMALLOC_NO_PRIVATE_NAMESPACE -o src/pages.sym.o src/pages.c
nm -a src/pages.sym.o | gawk -f include/jemalloc/internal/private_symbols.awk > src/pages.sym
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -DJEMALLOC_NO_PRIVATE_NAMESPACE -o src/prng.sym.o src/prng.c
nm -a src/prng.sym.o | gawk -f include/jemalloc/internal/private_symbols.awk > src/prng.sym
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -DJEMALLOC_NO_PRIVATE_NAMESPACE -o src/prof.sym.o src/prof.c
nm -a src/prof.sym.o | gawk -f include/jemalloc/internal/private_symbols.awk > src/prof.sym
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -DJEMALLOC_NO_PRIVATE_NAMESPACE -o src/rtree.sym.o src/rtree.c
nm -a src/rtree.sym.o | gawk -f include/jemalloc/internal/private_symbols.awk > src/rtree.sym
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -DJEMALLOC_NO_PRIVATE_NAMESPACE -o src/stats.sym.o src/stats.c
nm -a src/stats.sym.o | gawk -f include/jemalloc/internal/private_symbols.awk > src/stats.sym
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -DJEMALLOC_NO_PRIVATE_NAMESPACE -o src/sz.sym.o src/sz.c
nm -a src/sz.sym.o | gawk -f include/jemalloc/internal/private_symbols.awk > src/sz.sym
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -DJEMALLOC_NO_PRIVATE_NAMESPACE -o src/tcache.sym.o src/tcache.c
nm -a src/tcache.sym.o | gawk -f include/jemalloc/internal/private_symbols.awk > src/tcache.sym
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -DJEMALLOC_NO_PRIVATE_NAMESPACE -o src/ticker.sym.o src/ticker.c
nm -a src/ticker.sym.o | gawk -f include/jemalloc/internal/private_symbols.awk > src/ticker.sym
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -DJEMALLOC_NO_PRIVATE_NAMESPACE -o src/tsd.sym.o src/tsd.c
nm -a src/tsd.sym.o | gawk -f include/jemalloc/internal/private_symbols.awk > src/tsd.sym
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -DJEMALLOC_NO_PRIVATE_NAMESPACE -o src/witness.sym.o src/witness.c
nm -a src/witness.sym.o | gawk -f include/jemalloc/internal/private_symbols.awk > src/witness.sym
/bin/sh include/jemalloc/internal/private_namespace.sh src/jemalloc.sym src/arena.sym src/background_thread.sym src/base.sym src/bin.sym src/bitmap.sym src/ckh.sym src/ctl.sym src/div.sym src/extent.sym src/extent_dss.sym src/extent_mmap.sym src/hash.sym src/hooks.sym src/large.sym src/log.sym src/malloc_io.sym src/mutex.sym src/mutex_pool.sym src/nstime.sym src/pages.sym src/prng.sym src/prof.sym src/rtree.sym src/stats.sym src/sz.sym src/tcache.sym src/ticker.sym src/tsd.sym src/witness.sym > include/jemalloc/internal/private_namespace.gen.h
cp include/jemalloc/internal/private_namespace.gen.h include/jemalloc/internal/private_namespace.gen.h
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -o src/jemalloc.o src/jemalloc.c
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -o src/arena.o src/arena.c
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -o src/background_thread.o src/background_thread.c
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -o src/base.o src/base.c
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -o src/bin.o src/bin.c
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -o src/bitmap.o src/bitmap.c
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -o src/ckh.o src/ckh.c
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -o src/ctl.o src/ctl.c
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -o src/div.o src/div.c
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -o src/extent.o src/extent.c
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -o src/extent_dss.o src/extent_dss.c
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -o src/extent_mmap.o src/extent_mmap.c
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -o src/hash.o src/hash.c
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -o src/hooks.o src/hooks.c
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -o src/large.o src/large.c
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -o src/log.o src/log.c
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -o src/malloc_io.o src/malloc_io.c
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -o src/mutex.o src/mutex.c
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -o src/mutex_pool.o src/mutex_pool.c
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -o src/nstime.o src/nstime.c
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -o src/pages.o src/pages.c
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -o src/prng.o src/prng.c
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -o src/prof.o src/prof.c
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -o src/rtree.o src/rtree.c
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -o src/stats.o src/stats.c
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -o src/sz.o src/sz.c
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -o src/tcache.o src/tcache.c
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -o src/ticker.o src/ticker.c
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -o src/tsd.o src/tsd.c
gcc -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -c -D_GNU_SOURCE -D_REENTRANT -Iinclude -Iinclude -o src/witness.o src/witness.c
ar crus lib/libjemalloc.a src/jemalloc.o src/arena.o src/background_thread.o src/base.o src/bin.o src/bitmap.o src/ckh.o src/ctl.o src/div.o src/extent.o src/extent_dss.o src/extent_mmap.o src/hash.o src/hooks.o src/large.o src/log.o src/malloc_io.o src/mutex.o src/mutex_pool.o src/nstime.o src/pages.o src/prng.o src/prof.o src/rtree.o src/stats.o src/sz.o src/tcache.o src/ticker.o src/tsd.o src/witness.o
make[3]: 离开目录“/opt/redis-5.0.9/deps/jemalloc”
make[2]: 离开目录“/opt/redis-5.0.9/deps”
    CC adlist.o
    CC quicklist.o
    CC ae.o
    CC anet.o
    CC dict.o
    CC server.o
    CC sds.o
    CC zmalloc.o
    CC lzf_c.o
    CC lzf_d.o
    CC pqsort.o
    CC zipmap.o
    CC sha1.o
    CC ziplist.o
    CC release.o
    CC networking.o
    CC util.o
    CC object.o
    CC db.o
    CC replication.o
    CC rdb.o
    CC t_string.o
    CC t_list.o
    CC t_set.o
    CC t_zset.o
    CC t_hash.o
    CC config.o
    CC aof.o
    CC pubsub.o
    CC multi.o
    CC debug.o
    CC sort.o
    CC intset.o
    CC syncio.o
    CC cluster.o
    CC crc16.o
    CC endianconv.o
    CC slowlog.o
    CC scripting.o
    CC bio.o
    CC rio.o
    CC rand.o
    CC memtest.o
    CC crc64.o
    CC bitops.o
    CC sentinel.o
    CC notify.o
    CC setproctitle.o
    CC blocked.o
    CC hyperloglog.o
    CC latency.o
    CC sparkline.o
    CC redis-check-rdb.o
    CC redis-check-aof.o
    CC geo.o
    CC lazyfree.o
    CC module.o
    CC evict.o
    CC expire.o
    CC geohash.o
    CC geohash_helper.o
    CC childinfo.o
    CC defrag.o
    CC siphash.o
    CC rax.o
    CC t_stream.o
    CC listpack.o
    CC localtime.o
    CC lolwut.o
    CC lolwut5.o
    LINK redis-server
    INSTALL redis-sentinel
    CC redis-cli.o
    LINK redis-cli
    CC redis-benchmark.o
    LINK redis-benchmark
    INSTALL redis-check-rdb
    INSTALL redis-check-aof

Hint: It's a good idea to run 'make test' ;)

make[1]: 离开目录“/opt/redis-5.0.9/src”
```

### 指定目录安装

执行

```bash
make install PREFIX=/opt/redis
```

```bash
[root@TKZ redis-5.0.9]# pwd
/opt/redis-5.0.9
[root@TKZ redis-5.0.9]# make install PREFIX=/opt/redis
cd src && make install
make[1]: 进入目录“/opt/redis-5.0.9/src”
    CC Makefile.dep
make[1]: 离开目录“/opt/redis-5.0.9/src”
make[1]: 进入目录“/opt/redis-5.0.9/src”

Hint: It's a good idea to run 'make test' ;)

    INSTALL install
    INSTALL install
    INSTALL install
    INSTALL install
    INSTALL install
make[1]: 离开目录“/opt/redis-5.0.9/src”
[root@TKZ redis-5.0.9]# pwd
/opt/redis-5.0.9
[root@TKZ redis-5.0.9]# cd ..
[root@TKZ opt]# ll
drwxr-xr-x.  3 root  root         17 4月  24 13:49 redis
drwxrwxr-x.  6 root  root       4096 4月  17 18:45 redis-5.0.9
-rw-r--r--.  1 root  root    1986574 4月  24 00:14 redis-5.0.9.tar.gz
[root@TKZ opt]# cd redis
[root@TKZ redis]# ll
总用量 0
drwxr-xr-x. 2 root root 134 4月  24 13:49 bin
[root@TKZ redis]# cd bin/
[root@TKZ bin]# ll
总用量 32772
-rwxr-xr-x. 1 root root 4367000 4月  24 13:49 redis-benchmark
-rwxr-xr-x. 1 root root 8125296 4月  24 13:49 redis-check-aof
-rwxr-xr-x. 1 root root 8125296 4月  24 13:49 redis-check-rdb
-rwxr-xr-x. 1 root root 4807976 4月  24 13:49 redis-cli
lrwxrwxrwx. 1 root root      12 4月  24 13:49 redis-sentinel -> redis-server
-rwxr-xr-x. 1 root root 8125296 4月  24 13:49 redis-server
[root@TKZ bin]#
```
### 启动 Redis 服务

从 redis-5.0.9 源码目录中复制 redis.conf 到 redis 的安装目录

```bash
[root@TKZ bin]# cp /opt/redis-5.0.9/redis.conf /opt/redis/bin/
```

修改 redis.conf 文件，将 ```daemonize no``` 设置成 ```daemonize yes```

启动 redis 服务

```bash
[root@TKZ bin]# ./redis-server redis.conf
[root@TKZ bin]# ps -ef|grep redis
root     118338      1  0 14:32 ?        00:00:00 ./redis-server *:6379
root     118358  63071  0 14:33 pts/3    00:00:00 grep --color=auto redis
```

## 最后

欢迎大家关注我的关注我的公众号一起探讨研究技术。

![](https://img2020.cnblogs.com/blog/1326851/202003/1326851-20200307235900287-613114059.png)
