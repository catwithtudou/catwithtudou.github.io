# ElasticSearch基本概念


## 文档（Document）

- Elasticsearch 是面向文档的，**文档是所有可搜索数据的最小单位**
- 文档会被**序列化成 JSON 格式**，保存在 Elasticsearch 中
  - JSON 对象由字段构成
  - 每个字段都有对应的字段类型（字符串/数值/布尔值/日期/二进制/范围类型）
- **每个文档都有一个 UniqueID**（可指定也可自动生成）


### JSON 文档

- 一篇文档包含了一系列字段。类似数据库表中一条记录
- JSON 文档，格式灵活，不需要预先定义格式
  - 字段的类型可以指定或者由 Elasticsearch自动推算得到
  - 支持数组/支持嵌套

![](https://img.zhengyua.cn/img/20220303084355.png)

### 文档的元数据

元数据，**用于标注文档的相关信息**：

- `_index`：文档所属的索引名
- `_type`：文档所属的类型名
- `_id`：文档唯一ID
- `_source`：文档原始的 JSON 数据
- `_version`：文档的版本信息
- `_score`：相关性打分（搜索查询时）

> 在7.0之前，一个 Index 可以设置多个 Type。

![](https://img.zhengyua.cn/img/20220303084417.png)

## 索引

index（索引）是文档的容器，是一类文档的集合：

- **index**体现逻辑空间概念：每个索引都有自己的 Mapping 定义，用于定义包含文档的字段名和字段类型
- **Shard**体现物理空间概念：索引中的数据分散在 Shard 上

索引中的 Mapping 和 Setting：

- 前者定义**文档字段的类型**
- 后者定义**不同的数据分布**

> 索引在不同场景语意可能不同，比如除上面提到的索引语义外，当保存一个文档到 Elasticsearch 的过程也可以叫索引（动词）。


![](https://img.zhengyua.cn/img/20220303084621.png)

### 与 RDBMS 比较

|RDBMS|Elasticsearch|
|---|---|
|Table|Index(Type)|
|Row|Document|
|Column|Filed|
|Schema|Mapping|
|SQL|DSL|

区别主要在于：

- RDBMS：**事务性/Join**
- Elasticsearch：**Schemaless/相关性/高性能全文检索**

## REST API

![](https://img.zhengyua.cn/img/20220303085858.png)

![](https://img.zhengyua.cn/img/20220303092450.png)


下面是一些常用指令：

```shell
#查看索引相关信息
GET movies

#查看索引的文档总数
GET movies/_count

#查看前10条文档，了解文档格式
POST movies/_search
{
}

#_cat indices API
#查看indices
GET /_cat/indices/kibana*?v&s=index

#查看状态为绿的索引
GET /_cat/indices?v&health=green

#按照文档个数排序
GET /_cat/indices?v&s=docs.count:desc

#查看具体的字段
GET /_cat/indices/kibana*?pri&v&h=health,index,pri,rep,docs.count,mt

#How much memory is used per index?
GET /_cat/indices?v&h=i,tm&s=tm:desc
```

## 分布式架构

常见的分布式架构一般会带来以下好处：

- 高可用性：
  - **服务可用性**：允许有节点停止服务
  - **数据可用性**：部分节点丢失，不会丢失数据
- **可扩展性**：
  - 请求量提升或数据的不断增长（将数据分布到所有节点上）

Elasticsearch 的分布式架构可以做到：

- **存储的水平扩容**
- **提高系统可用性**，部分节点停止服务，整个集群的服务不受影响

涉及到 Elasticsearch 配置中：

- 不同的集群通过不同的名字区分，默认为`elasticsearch`
- 通过配置文件修改，或者在命令行中`-E cluster.name=catwithtudou`进行设定
- 一个集群可以有一个或多个节点

### 节点

节点即 Elasticsearch 的一个实例：

- **本质上就是一个 JAVA 进程**
- 一台机器上可以运行多个实例，但生产环境一般建议一台机器只运行一个 Elasticsearch 实例
  
涉及到相关配置中：

- 每个节点都有名字，可通过配置文件配置，或启动时`-E node.name=node1`指定
- 节点启动之后，**会分配一个 UID，保存在 data 目录下**

#### Master-eligible nodes&Master Node

- 每个节点在启动后，默认为一个  Master eligible 节点

> 可以设置`node.master:false`禁止

- **Master-eligible 节点可以参加选主流程，成为 Master 节点**

> 当第一个节点启动的时候，默认将自己选举成 Master 节点

- 每个节点上都保存了集群的状态，**只有 Master 节点才能修改集群的状态信息**，其中**集群状态（Cluster State）**，维护集群中的必要信息：
  - 所有的节点信息
  - 所有的索引和其相关的 Mapping 与 Settings 信息
  - 分片的路由信息

#### Data Node&Coordinating Node

- *Data Node*：可以保存数据的节点。**负责保存分片数据**。在数据扩展上起到重要作用
- Coordinating Node：**负责接收 Client 的请求，将请求分发到合适的节点，最终把结果汇集到一起**，每个节点默认都起到了该Node类型职责

#### Other Node Type

- *Hot&Warm Node*：不同硬件配置的 Data Node，用来实现 Hot&Warm 架构，降级集群部署的成本
- *Maching Learning Node*：负责跑机器学习的 Job，用来做异常检测
- *Tribe Node*：连接到不同的集群，并且支持将这些集群当成一个单独的集群处理

#### 配置节点类型

- 开发环境中一个节点可以承担多种角色
- 生产环境中，应该设置单一的角色的节点（dedicated node）

|节点类型|配置参数|默认值|
|----|----|----|
|master eligible|node.master|true|
|data|node.data|true|
|ingest|node.ingest|true|
|coordinating only|无|每个节点默认是|
|machine learning|node.ml|true（需要 enable x-pack）

### 分片

分片主要分为主分片和副本：

- Primary Shard：**用以解决数据水平扩展的问题**。通过主分片，可以将数据分布到集群内的所有节点上
  - 一个分片是一个运行的 Lucene 的实例
  - 主分片数在索引创建时指定，**后续不允许修改**，除非`Reindex`
- Replica Shard：**用以解决数据高可用的问题**。分片是主分片的拷贝：
  - 副本分片数，可以**动态调整**
  - 增加副本数，还可以一定程度上提高**服务的可用性**

![](https://img.zhengyua.cn/img/20220303102828.png)

#### 分片的设定

对于生产环境中分片的设定，**需要提前规划好容量**：

- 分片数设置过小：
  - 导致后续**无法增加节点实现水平扩展**
  - 单个分片的数据量太大，**导致数据分配耗时高**
- 分片数设置过大：
  - 影响搜索结果的**相关性打分**，影响统计结果的准确性
  - 单个节点上过多的分片，会导致**资源浪费，同时也会消耗性能**

> 7.0版本开始，默认主分片设置为 1，解决了 over-sharding 的问题。

### 查看集群的健康状况

使用`GET _cluster/health`可查询集群的健康状况：

- Green：主分片与副本都分配正常
- Yellow：主分片分配正常，副本分配不正常
- Red：有主分片未分配（如当服务器的磁盘容量超过85%时，创建了一个新的索引）

一般有以下常用指令：

```shell
GET _cat/nodes?v
GET /_nodes/es7_01,es7_02
GET /_cat/nodes?v
GET /_cat/nodes?v&h=id,ip,port,v,m


GET _cluster/health
GET _cluster/health?level=shards
GET /_cluster/health/kibana_sample_data_ecommerce
GET /_cluster/health/kibana_sample_data_ecommerce?level=shards

#### cluster state
GET /_cluster/state

#cluster get settings
GET /_cluster/settings
GET /_cluster/settings?include_defaults=true

GET _cat/shards
GET _cat/shards?h=index,shard,prirep,state,unassigned.reason
```

![](https://img.zhengyua.cn/img/20220303105951.png)


## 参考

https://time.geekbang.org/course/intro/100030501?tab=catalog
    "token" : "dogs",
      "start_offset" : 43,
      "end_offset" : 47,
      "type" : "word",
      "position" : 7
    },
    {
      "token" : "in",
      "start_offset" : 48,
      "end_offset" : 50,
      "type" : "word",
      "position" : 8
    },
    {
      "token" : "the",
      "start_offset" : 51,
      "end_offset" : 54,
      "type" : "word",
      "position" : 9
    },
    {
      "token" : "summer",
      "start_offset" : 55,
      "end_offset" : 61,
      "type" : "word",
      "position" : 10
    },
    {
      "token" : "evening.",
      "start_offset" : 62,
      "end_offset" : 70,
      "type" : "word",
      "position" : 11
    }
  ]
}
``` 

#### Keyword Analyzer

- 不分词，直接把输入当 term 输出

组成如下：

- Tokenizer:Keyword

#### Pattern Analyzer

- 通过正则表达式进行分词
- 默认是 \W+,非字符的符号进行分隔

组成如下：

- Tokenizer:Pattern
- TokenFilters:LowerCase&Stop

```json
GET _analyze
{
  "analyzer": "pattern",
  "text": "2 running Quick brown-foxes leap over lazy dogs in the summer evening."
}

{
  "tokens" : [
    {
      "token" : "2",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "word",
      "position" : 0
    },
    {
      "token" : "running",
      "start_offset" : 2,
      "end_offset" : 9,
      "type" : "word",
      "position" : 1
    },
    {
      "token" : "quick",
      "start_offset" : 10,
      "end_offset" : 15,
      "type" : "word",
      "position" : 2
    },
    {
      "token" : "brown",
      "start_offset" : 16,
      "end_offset" : 21,
      "type" : "word",
      "position" : 3
    },
    {
      "token" : "foxes",
      "start_offset" : 22,
      "end_offset" : 27,
      "type" : "word",
      "position" : 4
    },
    {
      "token" : "leap",
      "start_offset" : 28,
      "end_offset" : 32,
      "type" : "word",
      "position" : 5
    },
    {
      "token" : "over",
      "start_offset" : 33,
      "end_offset" : 37,
      "type" : "word",
      "position" : 6
    },
    {
      "token" : "lazy",
      "start_offset" : 38,
      "end_offset" : 42,
      "type" : "word",
      "position" : 7
    },
    {
      "token" : "dogs",
      "start_offset" : 43,
      "end_offset" : 47,
      "type" : "word",
      "position" : 8
    },
    {
      "token" : "in",
      "start_offset" : 48,
      "end_offset" : 50,
      "type" : "word",
      "position" : 9
    },
    {
      "token" : "the",
      "start_offset" : 51,
      "end_offset" : 54,
      "type" : "word",
      "position" : 10
    },
    {
      "token" : "summer",
      "start_offset" : 55,
      "end_offset" : 61,
      "type" : "word",
      "position" : 11
    },
    {
      "token" : "evening",
      "start_offset" : 62,
      "end_offset" : 69,
      "type" : "word",
      "position" : 12
    }
  ]
}
```

#### Language Analyzer

支持按不同国家语言进行分词。

```json
#english
GET _analyze
{
  "analyzer": "english",
  "text": "2 running Quick brown-foxes leap over lazy dogs in the summer evening."
}

{
  "tokens" : [
    {
      "token" : "2",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "<NUM>",
      "position" : 0
    },
    {
      "token" : "run",
      "start_offset" : 2,
      "end_offset" : 9,
      "type" : "<ALPHANUM>",
      "position" : 1
    },
    {
      "token" : "quick",
      "start_offset" : 10,
      "end_offset" : 15,
      "type" : "<ALPHANUM>",
      "position" : 2
    },
    {
      "token" : "brown",
      "start_offset" : 16,
      "end_offset" : 21,
      "type" : "<ALPHANUM>",
      "position" : 3
    },
    {
      "token" : "fox",
      "start_offset" : 22,
      "end_offset" : 27,
      "type" : "<ALPHANUM>",
      "position" : 4
    },
    {
      "token" : "leap",
      "start_offset" : 28,
      "end_offset" : 32,
      "type" : "<ALPHANUM>",
      "position" : 5
    },
    {
      "token" : "over",
      "start_offset" : 33,
      "end_offset" : 37,
      "type" : "<ALPHANUM>",
      "position" : 6
    },
    {
      "token" : "lazi",
      "start_offset" : 38,
      "end_offset" : 42,
      "type" : "<ALPHANUM>",
      "position" : 7
    },
    {
      "token" : "dog",
      "start_offset" : 43,
      "end_offset" : 47,
      "type" : "<ALPHANUM>",
      "position" : 8
    },
    {
      "token" : "summer",
      "start_offset" : 55,
      "end_offset" : 61,
      "type" : "<ALPHANUM>",
      "position" : 11
    },
    {
      "token" : "even",
      "start_offset" : 62,
      "end_offset" : 69,
      "type" : "<ALPHANUM>",
      "position" : 12
    }
  ]
}
```

#### 中文分词

中文分词难点较多，如需要切分成词、没有像英文自然的空格作为分隔、不同上下文分词理解不同等。

中文分词使用较为常见的分词器 ICU Analyzer：

- 需要安装 plugin
  - `Elasticsearch-plugin install analysis-icu`
- 提供了 Unicode 的支持，更好的支持亚洲语言

组成如下：

- CharacterFilters:Normalization
- Tokenizer:ICU Tokenizer
- TokenFilters:Normalization&Folding&Collation&Transform


```json
POST _analyze
{
  "analyzer": "icu_analyzer",
  "text": "他说的确实在理”"
}

{
  "tokens" : [
    {
      "token" : "他",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "<IDEOGRAPHIC>",
      "position" : 0
    },
    {
      "token" : "说的",
      "start_offset" : 1,
      "end_offset" : 3,
      "type" : "<IDEOGRAPHIC>",
      "position" : 1
    },
    {
      "token" : "确实",
      "start_offset" : 3,
      "end_offset" : 5,
      "type" : "<IDEOGRAPHIC>",
      "position" : 2
    },
    {
      "token" : "在",
      "start_offset" : 5,
      "end_offset" : 6,
      "type" : "<IDEOGRAPHIC>",
      "position" : 3
    },
    {
      "token" : "理",
      "start_offset" : 6,
      "end_offset" : 7,
      "type" : "<IDEOGRAPHIC>",
      "position" : 4
    }
  ]
}



POST _analyze
{
  "analyzer": "standard",
  "text": "他说的确实在理”"
}


{
  "tokens" : [
    {
      "token" : "他",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "<IDEOGRAPHIC>",
      "position" : 0
    },
    {
      "token" : "说",
      "start_offset" : 1,
      "end_offset" : 2,
      "type" : "<IDEOGRAPHIC>",
      "position" : 1
    },
    {
      "token" : "的",
      "start_offset" : 2,
      "end_offset" : 3,
      "type" : "<IDEOGRAPHIC>",
      "position" : 2
    },
    {
      "token" : "确",
      "start_offset" : 3,
      "end_offset" : 4,
      "type" : "<IDEOGRAPHIC>",
      "position" : 3
    },
    {
      "token" : "实",
      "start_offset" : 4,
      "end_offset" : 5,
      "type" : "<IDEOGRAPHIC>",
      "position" : 4
    },
    {
      "token" : "在",
      "start_offset" : 5,
      "end_offset" : 6,
      "type" : "<IDEOGRAPHIC>",
      "position" : 5
    },
    {
      "token" : "理",
      "start_offset" : 6,
      "end_offset" : 7,
      "type" : "<IDEOGRAPHIC>",
      "position" : 6
    }
  ]
}


POST _analyze
{
  "analyzer": "icu_analyzer",
  "text": "这个苹果不大好吃"
}
{
  "tokens" : [
    {
      "token" : "这个",
      "start_offset" : 0,
      "end_offset" : 2,
      "type" : "<IDEOGRAPHIC>",
      "position" : 0
    },
    {
      "token" : "苹果",
      "start_offset" : 2,
      "end_offset" : 4,
      "type" : "<IDEOGRAPHIC>",
      "position" : 1
    },
    {
      "token" : "不大",
      "start_offset" : 4,
      "end_offset" : 6,
      "type" : "<IDEOGRAPHIC>",
      "position" : 2
    },
    {
      "token" : "好吃",
      "start_offset" : 6,
      "end_offset" : 8,
      "type" : "<IDEOGRAPHIC>",
      "position" : 3
    }
  ]
}
```

更多的中文分词器：

- IK
  - 支持自定义词库，支持热更新分词字典
  - https://github.com/medcl/elasticsearch-analysis-ik
- THULAC
  - THU Lexucal Analyzer for Chinese，清华大学自然语言处理和社会人文计算实验室的一套中文分词器
  - https://github.com/microbun/elasticsearch-thulac-plugin


```json
POST _analyze
{
  "analyzer": "ik_smart",
  "text": "他说的确实在理”"
}

{
  "tokens" : [
    {
      "token" : "他",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "CN_CHAR",
      "position" : 0
    },
    {
      "token" : "说",
      "start_offset" : 1,
      "end_offset" : 2,
      "type" : "CN_CHAR",
      "position" : 1
    },
    {
      "token" : "的确",
      "start_offset" : 2,
      "end_offset" : 4,
      "type" : "CN_WORD",
      "position" : 2
    },
    {
      "token" : "实",
      "start_offset" : 4,
      "end_offset" : 5,
      "type" : "CN_CHAR",
      "position" : 3
    },
    {
      "token" : "在理",
      "start_offset" : 5,
      "end_offset" : 7,
      "type" : "CN_WORD",
      "position" : 4
    }
  ]
}


POST _analyze
{
  "analyzer": "ik_smart",
  "text": "这个苹果不大好吃"
}

{
  "tokens" : [
    {
      "token" : "这个",
      "start_offset" : 0,
      "end_offset" : 2,
      "type" : "CN_WORD",
      "position" : 0
    },
    {
      "token" : "苹果",
      "start_offset" : 2,
      "end_offset" : 4,
      "type" : "CN_WORD",
      "position" : 1
    },
    {
      "token" : "不大",
      "start_offset" : 4,
      "end_offset" : 6,
      "type" : "CN_WORD",
      "position" : 2
    },
    {
      "token" : "好吃",
      "start_offset" : 6,
      "end_offset" : 8,
      "type" : "CN_WORD",
      "position" : 3
    }
  ]
}



```

## 参考

https://time.geekbang.org/course/intro/100030501?tab=catalog



