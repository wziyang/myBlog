---
title: Elasticsearch-6.2.4插件开发一：插件种类介绍
tags: Elasticsearch
categories: Elasticsearch
---

# 插件开发
ES提供两种方式用于插件开发，一种是继承Plugin抽象类，一种是继承Plugin抽象类后再实现相关的插件接口。
# 插件种类
`ScriptPlugin`：脚本插件，ES默认的脚本语言是Painless，可自定义其他脚本语言，java、js等。

`AnalysisPlugin`：分析插件，可扩展新的分析器，标记器，标记过滤器或字符过滤器等。

`DiscoveryPlugin`：发现插件，使集群可以发现节点，如使建立在AWS上的集群可以发现节点。

`ClusterPlugin`：集群插件，增强对集群的管理，如控制分片位置。

`IngestPlugin`：摄取插件，增强节点的ingest功能，例如可以在ingest中通过tika解析ppt、pdf内容。

`MapperPlugin`：映射插件，可添加新的字段类型。

`SearchPlugin`：搜索插件，扩展搜索功能，如添加新的搜索类型，高亮类型等。

`RepositoryPlugin`：存储库插件，可添加新的存储库，如S3，Hadoop HDFS等。

`ActionPlugin`：API扩展插件，可扩展Http的API接口。

`NetworkPlugin`：网络插件，扩展ES底层的网络传输功能。

除了明确属于不同模块的插件接口外，继承抽象类Plugin可以为索引模块添加新的功能，如添加索引、搜索监听器，新的相似度算法。