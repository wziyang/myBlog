---
title: Elasticsearch-6.2.4插件开发二：插件的加载过程
tags: Elasticsearch
categories: Elasticsearch
---

## 从源码看插件的加载过程

插件是在节点（org.elasticsearch.node.Node）初始化的时候加载的，下面列举出部分关键代码，以搜索服务为例来介绍插件的加载过程：

```java
public Node(Environment environment) {
    this(environment, Collections.emptyList());
}
 
protected Node(final Environment environment, Collection<Class<? extends Plugin>> classpathPlugins) {
    ......
    // 插件服务从以下几个地方加载插件：plugins和modules文件夹、classpathPlugins集合（集合为空，可在开发时使用）
    this.pluginsService = new PluginsService(tmpSettings, environment.configFile(), 
        environment.modulesFile(), environment.pluginsFile(), 
        classpathPlugins);
    ......
    // 初始化search模块，可以看到实现SearchPlugin接口的插件类将会被过滤出来
    SearchModule searchModule = new SearchModule(settings, false,
        pluginsService.filterPlugins(SearchPlugin.class));
    ......
    // 各个模块整合成搜索服务searchService
    final SearchService searchService = newSearchService(clusterService, indicesService,
        threadPool, scriptModule.getScriptService(), bigArrays, searchModule.getFetchPhase(),
        responseCollectorService);
    ......
    modules.add(b -> {
        ......
        // 注入实例
        b.bind(SearchPhaseController.class).toInstance(new SearchPhaseController(settings,
            searchService::createReduceContext));
        ......
    }
    ......
}
```

接着我们来看pluginsService的构造函数：

```java
public PluginsService(Settings settings, Path configPath, Path modulesDirectory, Path pluginsDirectory, Collection<Class<? extends Plugin>> classpathPlugins) {
    ......
    // 以classpathPlugins的形式作为参考，可以看到需要继承Plugin抽象类
    for (Class<? extends Plugin> pluginClass : classpathPlugins) {
        ......
    }
}
```


可以看出，首先，我们要有一个类继承Plugin类并且某些服务需要实现相应的插件接口（旧版es只需要继承Plugin便可修改各个模块，相关接口现已弃用）才能被发现。

## 搜索插件的加载过程

接下来我们以搜索插件中的高亮部分为例，看看如何以插件的形式给es添加新的高亮类型。

首先是searchModule的构造函数：

```java
public SearchModule(Settings settings, boolean transportClient, List<SearchPlugin> plugins) {
    this.settings = settings;
    this.transportClient = transportClient;
    registerSuggesters(plugins);
    // 高亮加载
    highlighters = setupHighlighters(settings, plugins);
    registerScoreFunctions(plugins);
    registerQueryParsers(plugins);
    registerRescorers(plugins);
    registerSorts();
    registerValueFormats();
    registerSignificanceHeuristics(plugins);
    registerMovingAverageModels(plugins);
    registerAggregations(plugins);
    registerPipelineAggregations(plugins);
    registerFetchSubPhases(plugins);
    registerSearchExts(plugins);
    registerShapes();
}
```

可以看到构造函数中注册了各种搜索模块，而其中就有高亮的注册。

```java
private Map<String, Highlighter> setupHighlighters(Settings settings, List<SearchPlugin> plugins) {
    NamedRegistry<Highlighter> highlighters = new NamedRegistry<>("highlighter");
    highlighters.register("fvh",  new FastVectorHighlighter(settings));
    highlighters.register("plain", new PlainHighlighter());
    highlighters.register("unified", new UnifiedHighlighter());
    highlighters.extractAndRegister(plugins, SearchPlugin::getHighlighters);

    return unmodifiableMap(highlighters.getRegistry());
}
```

首先加载了3个系统自带的高亮类型，然后加载搜索插件中的高亮部分。