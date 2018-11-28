---
title: MySQL-5.7.22解压版相关操作（windows）
tags: 
- MySQL
categories: MySQL
copyright: true
---

# 安装

## 下载

mysql官网：https://dev.mysql.com/downloads/mysql/

<!-- more -->

## 解压

存放相关目录：G:\mysql\mysql-5.7.22-winx64

##  配置系统环境变量

添加：

`MYSQL_HOME=G:\mysql\mysql-5.7.22-winx64`

Path新增:

`MYSQL_HOME\bin`

## 配置my.ini

相关目录：G:\mysql\mysql-5.7.22-winx64下创建my.ini文件：

```bash
[client]
port=3306
default-character-set=utf8
 
[mysqld] 
# 设置为自己MYSQL的安装目录 
basedir=G:\mysql\mysql-5.7.22-winx64
# 设置为MYSQL的数据目录 
datadir=G:\mysql\mysql-5.7.22-winx64\data
port=3306
character_set_server=utf8
sql_mode=NO_ENGINE_SUBSTITUTION,NO_AUTO_CREATE_USER
#开启查询缓存
explicit_defaults_for_timestamp=true
#免密登录
#skip-grant-tables
#默认引擎
default-storage-engine=INNODB
log-error=G:\mysql\mysql-5.7.22-winx64\mysqlerror.log
log_error_verbosity=2
```

## 初始化**MySQL**

打开命令行操作：

`mysqld --defaults-file=G:\mysql\mysql-5.7.22-winx64\my.ini --initialize-insecure`

## 安装MySQL服务

`mysqld --install`

## 启动MySQL服务

`net start mysql`

## 进入MySQL

`mysql -uroot -p`

初次进入不需要密码。

## 修改root用户密码

`update mysql.user set authentication_string=password("新密码") where user="root";`

刷新权限表重置密码

`flush privileges;`

退出

 `quit`

## 重启MySQL服务，安装完成

命令行：

`net stop mysql`

`net start mysql`

# 忘记root密码

## 停止MySQL服务

`net stop mysql`

## 进入安全模式

`mysqld -nt --skip-grant-tables`

## 重新启动一个命令行

免密码进入数据库

`mysql -uroot -p`

## 修改密码

与前面操作相同。

修改完成后通过任务管理器关闭mysqld进程，重启服务

# 卸载MySQL

## 卸载服务

关闭MySQL服务后，卸载服务

`mysqld -remove`

卸载完成后删除目录

## 清理注册表

a、HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Services\Eventlog\Application\MySQL 目录删除

b、HKEY_LOCAL_MACHINE\SYSTEM\ControlSet002\Services\Eventlog\Application\MySQL 目录删除

c、HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Eventlog\Application\MySQL 目录删除

注册表中的ControlSet001、ControlSet002不一定是001和002，可能是ControlSet005、006之类。