---
title: Elasticsearch-6.2.4插件源码调试
tags: Elasticsearch
categories: Elasticsearch
---

正常es测试插件必须把插件打包好并放进plugins目录中，这样是很不方便的。而在源码中开发插件（将插件代码放入es源码里面）就可以不用每次都得把插件编译打包了，org.elasticsearch.node.Node（节点类，es启动时初始化节点的各个部分）的构造函数中就存在一个入口，可以在es启动时将我们的插件类加载进去。（ps：源码启动es可以看另一篇文章）

```java
public Node(Environment environment) {
	// 这里传入了一个空的集合，从下面可以看出这是插件的集合
	this(environment, Collections.emptyList());
}

protected Node(final Environment environment, Collection<Class<? extends Plugin>> classpathPlugins) {
 	// 节点初始化操作         
}
```

对于这个入口，可以写一个简单工厂类来加载我们的插件：

```java
public Node(Environment environment) {
    //this(environment, Collections.emptyList());
    this(environment, CreatePluginClassFactory.createPlugin( "test"));
}
```

```java
public class CreatePluginClassFactory {
    public static Collection<Class<? extends Plugin>> createPlugin(String ... pluginName) {
        Collection<Class<? extends Plugin>> result = new ArrayList<>();
        for(int i = 0; i < pluginName.length; i++) {
            switch (pluginName[i]) {
                case "test" :
                    // 继承了Plugin的插件类
                    result.add(TestPlugin.class);
                    break;
                default:
                    break;
            }
        }
        return result;
    }
}

```

这样将插件类加载进去后，在源码重启es便可以调试代码。