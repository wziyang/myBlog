---
title: Hadoop示例：wordcount
tags: 
- Hadoop
categories: Hadoop
copyright: true
---
附：Hadoop-2.9.0 fs shell：

http://hadoop.apache.org/docs/r2.9.0/hadoop-project-dist/hadoop-common/FileSystemShell.html

# 启动Hadoop

`./start-all.sh`

通过jps指令查看是否正常启动：

<!-- more -->

![img](https://wziyang.github.io/images/hadoop/jps.png)

# 在HDFS中创建input目录

`hadoop fs -mkdir /input`

创建结果：

`hadoop fs -ls /`

![img](https://wziyang.github.io/images/hadoop/fs_ls.png)

# 上传文件

输入文件：wordcount

```bash
hello world hadoop
hello hello hadoop world hello
hadoop world world hello hadoop
```

上传：

`hadoop fs -put wordcount /input`

# 调用wordcount示例

调用示例：将/input/wordcount作为输入文件，并将结果输出到/output目录中（输出目录会自动创建）

`hadoop jar /var/local/hadoop-2.9.0/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.9.0.jarwordcount /input/wordcount /output`
处理过程：

![img](https://wziyang.github.io/images/hadoop/wordcount处理.png)

# 查看输出结果

HDFS系统下多了个/output 和/tmp目录

/output目录下面有两个文件（_SUCCESS和part-r-00000），说明已经运行成功了

直接打开part-r-00000便可以查看结果

![img](https://wziyang.github.io/images/hadoop/fs_ls_output.png)

`hadoop fs -cat /output/part-r-00000`

![img](https://wziyang.github.io/images/hadoop/part-r-00000.png)

wordcount示例调试完成！