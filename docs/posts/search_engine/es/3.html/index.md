# ElasticSearchSearchAPI


## Search API 概览

主要包含两部分：

1. **URL Search**

在 URL 中使用查询参数

2. **Request Body Search**

使用 Elasticsearch 提供的，基于 JSON 格式的更加完备的`QueryDomainSpecificLanguage(DSL)`

### 指定查询的索引

- `/_search`：集群上所有的索引
- `/index1/_search`：index1
- `/index1,index2/_search`：index1和index2
- `/index*/_search`：以index开头的索引

### URL 查询

- 使用`q`，指定查询字符串
- `query strig syntax`，KV键值对

比如：

```shell
curl -XGET "http://localhost:9200/${index}/_search?q=${key}:${value}"
```

![](https://img.zhengyua.cn/img/202203062050129.png)

### Request Body 查询

```shell
#支持POST和GET
#'match_all'指返回所有的文档
curl -XGET "http://localhost:9200/${index}/_search" -H 'Content-Type:application/json` -d '{"query:{"match_all":{}}}'
```

![](https://img.zhengyua.cn/img/202203062053397.png)

### 搜索相关性 Relevance

**用户搜索关心的是搜索结果的相关性**，如是否可用找到所有有关的内容、是否有不相关的内容、文档的打分是否合理、结果排名是否满足需求等。

如不同的搜索需求：

- Web 搜索

![](https://img.zhengyua.cn/img/202203062039847.png)

- 电商搜索

![](https://img.zhengyua.cn/img/202203062040935.png)

衡量相关性（Information Retrieval）主要有以下标准衡量：

- **Precision**：尽可能返回较少的无关文档
- **Recall**：尽量返回较多的相关文档
- **Ranking**：是否能够按照相关度进行排序

> Precision 和 Recall 的计算类似机器学习中的模型评估
> 
> Precision = TP/TP+FP
> Recall = TP/TP+FN



## 参考

https://time.geekbang.org/course/intro/100030501?tab=catalog
