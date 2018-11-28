---
title: Elasticsearch-6.2.4插件开发四：分析插件开发
tags: 
- Elasticsearch
- 插件
- Analyzer
categories: Elasticsearch
copyright: true
---

# 分析器介绍

分析器内部分为`Analyzer`、`CharFilter`、`Tokenizer`、`TokenFilter`。

<!-- more-->

其中`Analyzer`由其他三部分组成，`Tokenizer`为必要部分，其他两个非必要。

数据进入分析器的流程：

![Analyzer Pipeline](https://wziyang.github.io/images/插件/Signatures.svg)

- Analyzer：分析器，对数据进行过滤、分词等操作

- CharFilter：字符过滤器，最开始处理数据，过滤指定字符，如html标签过滤

- Tokenizer：分词器，对数据进行分词操作，作为Analyzer的核心

- TokenFilter：分词过滤器，对分词进行过滤，如lowercase将分词中存在的大写字母过滤成小写


# 分词结果解析

```json
GET _analyze
{
  "analyzer": "standard",
  "text": ["测试test"]
}
```

```json
{
  "tokens": [
    {
      "token": "测",
      "start_offset": 0,
      "end_offset": 1,
      "type": "<IDEOGRAPHIC>",
      "position": 0
    },
    {
      "token": "试",
      "start_offset": 1,
      "end_offset": 2,
      "type": "<IDEOGRAPHIC>",
      "position": 1
    },
    {
      "token": "test",
      "start_offset": 2,
      "end_offset": 6,
      "type": "<ALPHANUM>",
      "position": 2
    }
  ]
}
```

文本经过分词后会返回一个分词数组：

- `token`表示分词

- `start_offset`和`end_offset`表示该分词在文本中的偏移位置

- `type`表示该分词的类型

- `position`表示分词的位置，注：match_phrase以`position`为标准

# 插件类

插件类需要继承Plugin类以及实现AnalysisPlugin接口，实现AnalysisPlugin接口可以重新其中的方法来提供指定的分析器：

```java
public interface AnalysisPlugin {
    default Map<String, AnalysisProvider<CharFilterFactory>> getCharFilters() {
        return emptyMap();
    }
    default Map<String, AnalysisProvider<TokenFilterFactory>> getTokenFilters() {
        return emptyMap();
    }
    default Map<String, AnalysisProvider<TokenizerFactory>> getTokenizers() {
        return emptyMap();
    }
    default Map<String, AnalysisProvider<AnalyzerProvider<? extends Analyzer>>> getAnalyzers() {
        return emptyMap();
    }
    ......
}
```

# Tokenizer开发

Tokenizer文件结构：

![分析插件uml](https://wziyang.github.io/images/插件/分析插件uml.png)

插件类中重写`getTokenizers` 方法，添加分词器工厂：

```java
public class TestAnalyzerPlugin extends Plugin implements AnalysisPlugin {
    @Override
    public Map<String, AnalysisProvider<TokenizerFactory>> getTokenizers() {
        Map<String, AnalysisProvider<TokenizerFactory>> tokenizers = new HashMap<>();
        tokenizers.put("test", TestTokenizerFactory::new);
        return tokenizers;
    }
}

```

定义TokenizerFactory：

```java
public class TestTokenizerFactory extends AbstractTokenizerFactory {
    // 自己定义配置类，非必须
    private AnalysisConfig config;

    public TestTokenizerFactory(IndexSettings indexSettings, Environment env, String name, Settings settings) {
        super(indexSettings, name, settings);
        // 可在设置tokenizer时读取相关配置
        this.config = new AnalysisConfig(settings);
    }

    @Override
    public Tokenizer create() {
        // 构建tokenizer
        return new TestTokenizer(this.config);
    }
}
```

配置类：

```java
public class AnalysisConfig {

    private boolean testConfig = true;

    public AnalysisConfig(Settings settings) {
        // 在setting中读取配置，如果没有则默认为true
        this.testConfig = settings.getAsBoolean("testConfig", true);
    }

    public boolean isTestConfig() {
        return testConfig;
    }
}
```

Tokenizer：

在此之前，先看看外部对分析器的调用：

```java
// 获取分析器的tokenStream，如果该分析器中只包括了Tokenizer，则tokenStream就是该分析器的Tokenizer
TokenStream tokenStream = analyzer.tokenStream(fieldName, contents))；
tokenStream.reset();
// 循环调用，每次获取一个分词
while (tokenStream.incrementToken()) {
    // 得到分词的偏移量
    OffsetAttribute attr = tokenStream.getAttribute(OffsetAttribute.class);
}
tokenStream.end();
```

定义Tokenizer：

```java
public class TestTokenizer extends Tokenizer {
    // 分词结果
    private final CharTermAttribute termAtt = addAttribute(CharTermAttribute.class);
    // 偏移量
    private final OffsetAttribute offsetAtt = addAttribute(OffsetAttribute.class);
    // 位置
    private final PositionIncrementAttribute posIncrAtt = addAttribute(PositionIncrementAttribute.class);
    
    private AnalysisConfig config;

    public TestTokenizer(AnalysisConfig config) {
        this.config = config;
    }

    @Override
    public void reset() throws IOException {
        // 分词前执行，如果有设置变量需要在这里清理资源
        super.reset();
    }

    @Override
    public void end() throws IOException {
        super.end();
    }

    /**
    * 将循环调用改方法
    */
    @Override
    public boolean incrementToken() throws IOException {
        // 清理上一次分词的数据
        clearAttributes();
		// 从Reader中读出数据：input.read()，可存在局部变量之中
        // 存储分词信息
        termAtt.append(term);
        offsetAtt.setOffset(startOffset, endOffset);
        posIncrAtt.setPositionIncrement(offset);
        // 返回true表示分词还没结束
        return true;
    }
}

```

