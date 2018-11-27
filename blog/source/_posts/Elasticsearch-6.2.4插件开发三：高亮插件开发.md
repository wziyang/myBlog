---
title: Elasticsearch-6.2.4插件开发三：高亮插件开发
tags: Elasticsearch
categories: Elasticsearch
---

首先，我们需要继承`Plugin`类使插件可以被es初始化时候加载，然后根据实际功能具体实现不同的插件接口，重写相关方法。高亮是搜索阶段的处理，我们可以看看`SearchPlugin`里的部分代码：

```java
public interface SearchPlugin {

    default List<ScoreFunctionSpec<?>> getScoreFunctions() {
        return emptyList();
    }
    default List<SearchExtensionSpec<SignificanceHeuristic, SignificanceHeuristicParser>> getSignificanceHeuristics() {
        return emptyList();
    }
    default List<SearchExtensionSpec<MovAvgModel, MovAvgModel.AbstractModelParser>> getMovingAverageModels() {
        return emptyList();
    }
    default List<FetchSubPhase> getFetchSubPhases(FetchPhaseConstructionContext context) {
        return emptyList();
    }
    default List<SearchExtSpec<?>> getSearchExts() {
        return emptyList();
    }
    // 高亮
    default Map<String, Highlighter> getHighlighters() {
        return emptyMap();
    }
    default List<SuggesterSpec<?>> getSuggesters() {
        return emptyList();
    }
    default List<QuerySpec<?>> getQueries() {
        return emptyList();
    }
    default List<AggregationSpec> getAggregations() {
        return emptyList();
    }
    default List<PipelineAggregationSpec> getPipelineAggregations() {
        return emptyList();
    }
    default List<RescorerSpec<?>> getRescorers() {
        return emptyList();
    }
	// 以下省略
}
```

`SearchPlugin`下的方法包括了搜索阶段的各种处理，其中就有我们需要的`getHighlighters`方法，我们只要实现`SearchPlugin`接口，重写`getHighlighter`方法即可。

高亮插件文件结构：

![高亮插件uml](https://wziyang.github.io/images/插件/高亮插件uml.png)

```java
public class TestHighlightSearchPlugin extends Plugin implements SearchPlugin {

    @Override
    public Map<String, Highlighter> getHighlighters() {
        // 返回高亮器
        return Collections.singletonMap(TestHighlighter.NAME, new TestHighlighter());
    }
}
```

需要返回一个map，key-value具体对应了高亮器的唯一标识，和一个实现了Highlighter接口的高亮器。

```java
public interface Highlighter {

    HighlightField highlight(HighlighterContext highlighterContext);

    boolean canHighlight(FieldMapper fieldMapper);
}
```

`Highlighter`接口里面就很简单了，只有两个方法，其中`canHighlight`中可以定义高亮器的使用限制条件，就像fvh需要设置`"term_vector": "with_positions_offsets"`一样，不需要限制的话默认返回true就行；而重点就是`highlight`方法了，可以在`highlighterContext`中拿到搜索返回的内容、分析器、分词等，自己定义高亮逻辑。