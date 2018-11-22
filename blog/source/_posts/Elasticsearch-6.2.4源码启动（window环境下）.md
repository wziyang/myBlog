---
title: Elasticsearch-6.2.4源码启动（window环境下）
tags: Elasticsearch
categories: Elasticsearch
---

# 1. 获取源码和可运行的es

1.1 首先从github上把es的源码拉下来并切换分支到v6.2.4版本（ps：es不同版本源码启动过程差异可能较大）。

```
git clone https://github.com/elastic/elasticsearch.git
```

```
git checkout v6.2.4
```

（本篇博客命令执行皆使用git bash）

1.2 到官网下载可运行的es

# 2. 编译源码

es是一个gradle项目，导入idea前需执行` gradle idea `命令，不然会报错。

![img](http://www.wziyang.cn/wordpress/wp-content/uploads/2018/09/gradle_idea%E6%8A%A5%E9%94%99.png)

使用项目自带的gradlew编译es源码

./gradlew idea

./gradlew build

（IDE是eclipse的话则将命令中idea替换成eclipse）

编译完成后便可用idea打开项目。

 

# 3. 运行es

如何从源码运行es？

我们先参考官网下载的es中，bin目录下的启动脚本elasticsearch，elasticsearch-env

```
(只列出关键部分，其余省略)
SCRIPT="$0"
ES_HOME=`dirname "$SCRIPT"`
ES_PATH_CONF="$ES_HOME"/config
......
"$JAVA" \
$ES_JAVA_OPTS \
-Des.path.home="$ES_HOME" \
-Des.path.conf="$ES_PATH_CONF" \
-cp "$ES_CLASSPATH" \
org.elasticsearch.bootstrap.Elasticsearch \
......
```

可见，es的启动入口为org.elasticsearch.bootstrap.Elasticsearch；

es启动需要配置两个启动参数es.path.home和es.path.conf

其中es.path.home是es服务的路径，即es项目中server目录下，es.path.conf为配置文件目录，在server目录下创建config文件夹作为配置文件目录，并直接从官网的es中copy三个配置文件（此处不详解文件具体内容）。

![img](http://www.wziyang.cn/wordpress/wp-content/uploads/2018/09/config目录下文件.png)

打开idea配置两个启动参数（直接绝对路径即可）

```
-Des.path.conf=E:\......\elasticsearch\server\config
-Des.path.home=E:\......\elasticsearch\server
```

在idea顶层菜单打开Run>Edit Configurations，配置VM options

![img](http://www.wziyang.cn/wordpress/wp-content/uploads/2018/09/配置两个路径.png)

接下来我们启动试一试：src>main>java>org>elasticsearch>bootstrap>Elasticsearch

直接执行main方法，发现启动失败，出现几个异常：

第一个

![img](http://www.wziyang.cn/wordpress/wp-content/uploads/2018/09/没有plugin目录.png)

原因是es启动时会去扫描主目录下的plugins目录，加载es插件（插件方面将在其他文章说明），所以要创建plugins目录。

第二个

![img](http://www.wziyang.cn/wordpress/wp-content/uploads/2018/09/权限问题.png)

原因是用户没有操作目录的权限

直接在server目录下建立java.policy文件：

```
grant codeBase "file:${{java.ext.dirs}}/*" {
        permission java.security.AllPermission;
        permission java.lang.RuntimePermission "createClassLoader";

};
grant {
        permission java.security.AllPermission;
        permission java.lang.RuntimePermission "createClassLoader";
};
```

VM options的配置增加：

```
-Des.path.conf=E:\......\elasticsearch\server\config
-Des.path.home=E:\......\elasticsearch\server
-Djava.security.policy=E:\......\elasticsearch\server\java.policy
```

第三个

![img](http://www.wziyang.cn/wordpress/wp-content/uploads/2018/09/module缺失.png)

原因是相关模块的缺失，es加载为空，直接把官网的es中的modules文件夹整个复制到server目录下就行。

第四个

![img](http://www.wziyang.cn/wordpress/wp-content/uploads/2018/09/类没有.png)

ExtendedPluginsClassLoader这个类并没有编译出来

至于原因，我们可以在单元测试类BootstrapForTesting中看到这一段：

![img](http://www.wziyang.cn/wordpress/wp-content/uploads/2018/09/classloader没编译.png) 原来eclipse和idea并不会打包内置的依赖，即传递依赖：

![img](http://www.wziyang.cn/wordpress/wp-content/uploads/2018/09/compileonly编译.png)

可以看到plugin-classloader的编译类型是compileOnly和testRuntime，只在编译阶段编译，在运行阶段不会将这个依赖包括进来，idea是默认不会运行时不包括传递依赖的，可以在这里勾选设置启动将包括该类传递依赖

![img](http://www.wziyang.cn/wordpress/wp-content/uploads/2018/09/勾选设置.png)

所有问题解决后便可以顺利启动了。

![img](http://www.wziyang.cn/wordpress/wp-content/uploads/2018/09/启动成功.png)