---
layout: post
title: docker mysql5.7主从数据库搭建
categories: [http, https, mysql, docker]
description: docker mysql5.7主从数据库搭建
keywords: mysql, 主从
---


## 1、容器启动

### 启动主库
```
docker run -d -p 3306:3306 --name mysql-master -v ~/Workspace/mysql/master/conf/my.cnf:/etc/mysql/my.cnf -e MYSQL_ROOT_PASSWORD=123456 mysql:5.7.18
```

### 启动从库-1
```
docker run -d -p 3316:3306 --name mysql-slave-1 -v ~/Workspace/mysql/slave1/conf/my.cnf:/etc/mysql/my.cnf -e MYSQL_ROOT_PASSWORD=123456 mysql:5.7.18
```

### 启动从库-2
```
docker run -d -p 3326:3306 --name mysql-slave-2 -v ~/Workspace/mysql/slave2/conf/my.cnf:/etc/mysql/my.cnf -e MYSQL_ROOT_PASSWORD=123456 mysql:5.7.18
```

## 2、主库(mysql-master)配置

- 数据库配置文件 `my.cof` 内容

	```
	!includedir /etc/mysql/conf.d/
	!includedir /etc/mysql/mysql.conf.d/
	[mysql]
	default-character-set = utf8
	[mysql.server]
	default-character-set = utf8
	[mysqld_safe]
	default-character-set = utf8
	[client]
	default-character-set = utf8
	[mysqld]
	lower_case_table_names=1
	character_set_server=utf8
	skip-name-resolve
	init_connect='SET NAMES utf8'
	server-id=100
	log-bin=mysql-bin

	```

- 进入主库容器内部 `docker exec -it mysql-master /bin/bash`
- 连接数据库 `mysql -h 127.0.0.1 -u root -p`，然后输入密码 `123456`
- 创建数据同步账户 `CREATE USER 'slave'@'%' IDENTIFIED BY '123456';`， 分配权限 `GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'slave'@'%';`
- 查看主库状态 `show master status;`，记住File和Position的值，配置从库要用的。

	```
	+------------------+----------+--------------+------------------+-------------------+
	| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
	+------------------+----------+--------------+------------------+-------------------+
	| mysql-bin.000003 |      609 |              |                  |                   |
	+------------------+----------+--------------+------------------+-------------------+
	```

## 3、从库(mysql-slave1)配置

- 数据库配置文件 `my.cof` 内容

	```
	!includedir /etc/mysql/conf.d/
	!includedir /etc/mysql/mysql.conf.d/
	[mysql]
	default-character-set = utf8
	[mysql.server]
	default-character-set = utf8
	[mysqld_safe]
	default-character-set = utf8
	[client]
	default-character-set = utf8
	[mysqld]
	lower_case_table_names=1
	character_set_server=utf8
	skip-name-resolve
	init_connect='SET NAMES utf8'
	server-id=101  
	log-bin=mysql-slave-bin   
	relay_log=edu-mysql-relay-bin
	```

- 进入主库容器内部 `docker exec -it mysql-master /bin/bash`
- 连接数据库 `mysql -h 127.0.0.1 -u root -p`，然后输入密码 `123456`
- 执行命令 `change master to master_host='172.17.0.2', master_user='slave', master_password='123456', master_port=3306, master_log_file='mysql-bin.000003', master_log_pos= 609, master_connect_retry=30;` 参数说明：

	```
	master_host ：Master的地址，指的是容器的独立ip,可以通过 docker inspect --format='{{.NetworkSettings.IPAddress}}' 容器名称|容器id查询容器的ip
	master_port：Master的端口号，指的是容器的端口号
	master_user：用于数据同步的用户
	master_password：用于同步的用户的密码
	master_log_file：指定 Slave 从哪个日志文件开始复制数据，即上文中提到的 File 字段的值
	master_log_pos：从哪个 Position 开始读，即上文中提到的 Position 字段的值
	master_connect_retry：如果连接失败，重试的时间间隔，单位是秒，默认是60秒
	```
- 查看从库状态 `show slave status`
- 开启主从复制过程 `start slave`
