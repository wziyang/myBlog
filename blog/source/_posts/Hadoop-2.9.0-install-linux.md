---
title: Hadoop-2.9.0，linux单机安装
tags: 
- Hadoop
- linux
categories: Hadoop
copyright: true
---

# 系统环境

CenterOS7.3，jdk1.7

<!-- more -->

# Hadoop安装

## 下载Hadoop

`wget http://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/common/hadoop-2.9.0/hadoop-2.9.0.tar.gz`

`mv hadoop-2.9.0.tar.gz /var/local`

`tar -xzvf hadoop-2.9.0.tar.gz`

##  修改相关配置文件

### 环境变量配置

`vim /etc/profile`

```bash
#------------------Hadoop--------------------
HADOOP_HOME=/var/local/hadoop-2.9.0
HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
PATH=$PATH:$HADOOP_HOME/bin
export HADOOP_HOME HADOOP_CONF_DIR PATH
```

`source /etc/profile`

### Hadoop配置文件修改

hadoop-2.9.0/etc/hadoop/core-site.xml

```xml
<configuration>  
    <property>  
        <name>fs.defaultFS</name>  
        <value>hdfs://localhost:9998</value>  
    </property>  
    <property>  
        <name>hadoop.tmp.dir</name>  
        <value>/opt/hadoop/tmp</value>  
    </property>  
</configuration>
```

hadoop-2.9.0/etc/hadoop/hdfs-site.xml

```xml
<configuration>  
    <property>  
        <name>dfs.name.dir</name>  
        <value>/var/local/hadoop-2.9.0/hdfs/name</value>  
    </property>  
    <property>  
        <name>dfs.data.dir</name>  
        <value>/var/local/hadoop-2.9.0/hdfs/data</value>  
    </property>    
    <property>  
        <name>dfs.replication</name>  
        <value>1</value>  
    </property> 
</configuration>
```

hadoop-2.9.0/etc/hadoop/mapred-site.xml

```xml
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

hadoop-2.9.0/etc/hadoop/yarn-site.xml

```xml
<configuration> 
    <property>
        <name>yarn.nodemanager.aux-services</name>
	<value>mapreduce_shuffle</value>
    </property>
    <property>
	<name>yarn.nodemanager.vmem-check-enabled</name>
	<value>false</value>
    </property>
</configuration>
```

hadoop-2.9.0/etc/hadoop/hadoop-env.sh

```bash
export JAVA_HOME=/usr/local/lib/jdk/jdk1.7.0_80
```

## 启动Hadoop

###  格式化Hadoop

`hadoop namenode -format`

###  启动

`./start-all.sh`

通过jps指令查看是否正常启动：

![img](https://wziyang.github.io/images/hadoop/jps.png)

###  启动结果

访问http://{server-ip}:8088

![img](https://wziyang.github.io/images/hadoop/hadoop启动结果.jpg)

## 关闭Hadoop

`./stop-all.sh`

至此，Hadoop安装完成！
