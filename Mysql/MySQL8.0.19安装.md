

```bash
[root@TKZ mysql]# ll
总用量 1494740
-rw-r--r--. 1 root root  765306880 12月 10 18:49 mysql-8.0.19-1.el7.x86_64.rpm-bundle.tar
-rw-r--r--. 1 7155 31415  43126424 12月 10 18:44 mysql-community-client-8.0.19-1.el7.x86_64.rpm
-rw-r--r--. 1 7155 31415    619248 12月 10 18:44 mysql-community-common-8.0.19-1.el7.x86_64.rpm
-rw-r--r--. 1 7155 31415   7421828 12月 10 18:44 mysql-community-devel-8.0.19-1.el7.x86_64.rpm
-rw-r--r--. 1 7155 31415  23691824 12月 10 18:44 mysql-community-embedded-compat-8.0.19-1.el7.x86_64.rpm
-rw-r--r--. 1 7155 31415   4212908 12月 10 18:44 mysql-community-libs-8.0.19-1.el7.x86_64.rpm
-rw-r--r--. 1 7155 31415   1365572 12月 10 18:44 mysql-community-libs-compat-8.0.19-1.el7.x86_64.rpm
-rw-r--r--. 1 7155 31415 457272180 12月 10 18:45 mysql-community-server-8.0.19-1.el7.x86_64.rpm
-rw-r--r--. 1 7155 31415 227581052 12月 10 18:46 mysql-community-test-8.0.19-1.el7.x86_64.rpm
[root@TKZ mysql]# rpm -ivh mysql-community-common-8.0.19-1.el7.x86_64.rpm
警告：mysql-community-common-8.0.19-1.el7.x86_64.rpm: 头V3 DSA/SHA1 Signature, 密钥 ID 5072e1f5: NOKEY
准备中...                          ################################# [100%]
正在升级/安装...
   1:mysql-community-common-8.0.19-1.e################################# [100%]
[root@TKZ mysql]# rpm -ivh mysql-community-libs-8.0.19-1.el7.x86_64.rpm
警告：mysql-community-libs-8.0.19-1.el7.x86_64.rpm: 头V3 DSA/SHA1 Signature, 密钥 ID 5072e1f5: NOKEY
准备中...                          ################################# [100%]
正在升级/安装...
   1:mysql-community-libs-8.0.19-1.el7################################# [100%]
[root@TKZ mysql]# rpm -ivh mysql-community-client-8.0.19-1.el7.x86_64.rpm
警告：mysql-community-client-8.0.19-1.el7.x86_64.rpm: 头V3 DSA/SHA1 Signature, 密钥 ID 5072e1f5: NOKEY
准备中...                          ################################# [100%]
正在升级/安装...
   1:mysql-community-client-8.0.19-1.e################################# [100%]
[root@TKZ mysql]# rpm -ivh mysql-community-server-8.0.19-1.el7.x86_64.rpm
警告：mysql-community-server-8.0.19-1.el7.x86_64.rpm: 头V3 DSA/SHA1 Signature, 密钥 ID 5072e1f5: NOKEY
准备中...                          ################################# [100%]
正在升级/安装...
   1:mysql-community-server-8.0.19-1.e################################# [100%]
[root@TKZ mysql]#
[root@TKZ mysql]# mysqld --initialize;
[root@TKZ mysql]# chown -R mysql:mysql /var/lib/mysql
[root@TKZ mysql]# systemctl start mysqld.service;
[root@TKZ mysql]# systemctl enable mysqld;
[root@TKZ mysql]# cat /var/log/mysqld.log | grep password
2020-04-18T13:20:34.942349Z 5 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: _Vv1dU6eP?9+
[root@TKZ mysql]# mysql
mysql                      mysqlcheck                 mysqld-debug               mysqldumpslow              mysql_secure_installation  mysql_ssl_rsa_setup
mysqladmin                 mysql_config_editor        mysqld_pre_systemd         mysqlimport                mysqlshow                  mysql_tzinfo_to_sql
mysqlbinlog                mysqld                     mysqldump                  mysqlpump                  mysqlslap                  mysql_upgrade
[root@TKZ mysql]# mysql -uroot -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 9
Server version: 8.0.19

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.
mysql> alter user 'root'@'localhost' identified by '12345678';
Query OK, 0 rows affected (0.03 sec)

mysql> exit;
Bye
[root@TKZ mysql]# mysql -uroot -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 10
Server version: 8.0.19 MySQL Community Server - GPL

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.01 sec)

mysql> use mysql
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+---------------------------+
| Tables_in_mysql           |
+---------------------------+
| columns_priv              |
| component                 |
| db                        |
| default_roles             |
| engine_cost               |
| func                      |
| general_log               |
| global_grants             |
| gtid_executed             |
| help_category             |
| help_keyword              |
| help_relation             |
| help_topic                |
| innodb_index_stats        |
| innodb_table_stats        |
| password_history          |
| plugin                    |
| procs_priv                |
| proxies_priv              |
| role_edges                |
| server_cost               |
| servers                   |
| slave_master_info         |
| slave_relay_log_info      |
| slave_worker_info         |
| slow_log                  |
| tables_priv               |
| time_zone                 |
| time_zone_leap_second     |
| time_zone_name            |
| time_zone_transition      |
| time_zone_transition_type |
| user                      |
+---------------------------+
33 rows in set (0.01 sec)

mysql> select host,user from user;
+-----------+------------------+
| host      | user             |
+-----------+------------------+
| localhost | mysql.infoschema |
| localhost | mysql.session    |
| localhost | mysql.sys        |
| localhost | root             |
+-----------+------------------+
4 rows in set (0.00 sec)

mysql> update user set host = '%' where user ='root';
Query OK, 1 row affected (0.12 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> select host,user from user;
+-----------+------------------+
| host      | user             |
+-----------+------------------+
| %         | root             |
| localhost | mysql.infoschema |
| localhost | mysql.session    |
| localhost | mysql.sys        |
+-----------+------------------+
4 rows in set (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.01 sec)

mysql>

```
