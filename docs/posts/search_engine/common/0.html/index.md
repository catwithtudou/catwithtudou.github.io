# （WIP）《这就是搜索引擎》阅读笔记


## 第一章 搜索引擎及其技术架构

### 搜索引擎为何重要

搜索是**目前解决信息过载的相对有效方式**，在没有更有效的替代解决方式出来之前，搜索引擎作为互联网网站和应用的入口及处于行业制高点的重要地位只会逐步加强。

### 搜索引擎发展史

- 分类目录搜索引擎：人工分类
- 第一代文本检索搜索引擎：简单检索
- 第二代引入链接分析技术：改善搜索结果质量
- 第三代以用户为中心：尝试用户个性化需求

### 搜索引擎的3个目标

- 更全（如提高网络爬虫技术）
- 更快（如索引、缓存等相关技术）
- 更准（如排序技术、链接分析技术等）

### 搜索引擎的3个核心问题

- 用户真正的需求是什么
- 哪些信息是和用户需求真正相关的
- 哪些信息是用户可以信赖的

### 搜索引擎的技术架构

![s20325802262022](https://img.zhengyua.cn/s20325802262022.png)

## 第二章 网络爬虫

### 通用爬虫框架

![s23185302262022](https://img.zhengyua.cn/s23185302262022.png)

### 优秀爬虫的特性

- 高性能
- 可扩展性
- 健壮性
- 友好性

### 爬虫质量的评价标准

- 抓取网页覆盖率
- 抓取网页时新性
- 抓取网页重要性

通盘考虑以上3个因素，可以将目前爬虫研发的目标简单描述如下：在资源有限的情况下，既然搜索引擎只能抓取互联网现存网页的一部分，那么就尽可能选择比较重要的那部分页面来索引；对于已经抓取到的网页，尽可能快地更新其内容，使得索引网页和互联网对应页面内容同步更新；在此基础上，尽可能扩大抓取范围，抓取到更多以前无法发现的网页。

### 抓取策略

爬虫的抓取策略有很多种，但不论方法如何，其基本目标一致：优先选择重要网页进行抓取。而其网页的重要性可以选择不同的方法进行评判（大部分按网页的流行性来定义）：

- 宽度优先遍历策略（Breath First）
- 非完全PageRank策略
- OCIP策略
- 大战优先策略

#### 宽度优先遍历策略

![](https://img.zhengyua.cn/img/20220226234440.png)

该策略的核心思想是**将新下载网页包含的链接直接追加到待抓取URL队列末尾**。

> 实验表明这种策略效果很好，虽然看似机械，但实际上的网页抓取顺序基本是按照网页的重要性排序的。原因在于实际上宽度优先遍历策略隐含了一些网页优先级假设.

#### 非完全PageRank策略（Partial PageRank）

PageRank是一种著名的链接分析算法，可以用来衡量网页的重要性。

该策略基本思路为**对于已经下载的网页，加上待抓取URL队列中的URL一起，形成网页集合，在此集合内进行PageRank计算，计算完成后，将待抓取URL队列里的网页按照PageRank得分由高到低排序，形成的序列就是爬虫接下来应该依次抓取的URL列表。**

> 考虑到非完全PageRank赋予这些新抽取出来但是又没有PageRank值的网页，可以通过一个临时PageRank值，将这个网页的所有入链传导的PageRank值汇总，作为临时PageRank值，若该值比目前队列中已计算的PageRank高的话那么优先下载这个URL。

![](https://img.zhengyua.cn/img/20220227095543.png)

#### OCIP策略（Online Page Importance Computation）

OCIP的字面含义是“在线页面重要性计算”，可以将其看做是一种改进的PageRank算法。

该策略基本思路为**假设为每个网页都给予相同的“现金”，每当下载了某个页面P后，P将自己拥有的“现金”平均分配给页面中包含的链接页面。而对于待抓取URL队列的网页中根据其手头拥有的现金金额多少排序，优先下载现金最充裕的网页。**

与非完全PageRank策略思路基本一致，区别在于：

- 该策略不需要迭代过程，增加了计算速度，更利于实时计算使用
- PageRank在计算时，存在向无链接关系网页的远程跳转过程，而OCIP没有这一计算因子；

实验结果表明，OCIP是种较好的重要性衡量策略，**效果略优于宽度优先遍历策略。**

#### 大站优先策略（Larger Sites First）

该策略基本思路为**以网站为单位来衡量网页重要性，对于待抓取URL队列中的网页，根据所属网站归类，如果哪个网站等待下载的页面最多，则优先下载这些链接。** 即倾向于优先下载大型网站。

这个思路虽然简单，但是有一定依据。实验表明这个算法效果也要略优于宽度优先遍历策略。

### 网页更新策略

为了更优的用户体验，对于爬虫已经抓取过的页面，还需要负责保持其内容和互联网页面内容的同步，这就取决于爬虫所采用的网页更新策略。

该策略核心点就在于重新抓取之前已经下载过的网页的时机，且尽可能使得本地下载网页和互联网原始页面内容保持一致。一般策略如下：

- 历史参考策略
- 用户体验策略
- 聚类抽样策略

#### 历史参考策略

该策略主要建立在以下前提：

- **过去频繁更新的网页，那么将来也会频繁更新。**

所以预估某个网页何时进行更新可参考其历史更新情况。

> 这里预估就需要建模，用模型来预测发生变化的时机，如泊松过程。

#### 用户体验策略

该策略判断一个网页更新时机取决于这个网页的内容变化所带来搜索质量的变化（往往采用搜索结果排名的变化来衡量），影响越大的网页，则应该越快更新。

用户体验策略**保存网页的多个历史版本，并根据过去每次内容变化对搜索质量的影响，得出一个平均值，以此作为判断爬虫重抓该网页时机的参考依据，对于影响越厉害的网页，则越优先调度重新抓取。**

#### 聚类抽样策略

该策略就是为了解决严重依赖历史更新信息的局限性。认为**网页具有一些属性，根据这些属性可以预测其更新周期，具有相似属性的网页其更新周期也是类似的。** 根据这些属性将网页归类，同一类别内的网页具有相同的更新频率。为了计算某个类别的更新周期，只需对类别内网页进行采样，以这些被采样网页的更新周期作为类别内所有其他网页的更新周期。

![](https://img.zhengyua.cn/img/20220227102547.png)

> 相关实验表明，聚类抽样策略效果好于前述两种更新策略，但是对以亿计的网页进行聚类，其难度也是非常巨大的。

### 暗网抓取

所谓暗网，是指目前搜索引擎爬虫按照常规方式很难抓取到的互联网页面。为了能够对暗网数据进行索引，需要研发与常规爬虫机制不同的系统，这类爬虫被称做暗网爬虫。

- 暗网爬虫的目的是**将暗网数据从数据库中挖掘出来，并将其加入搜索引擎的索引**，增加其搜索时的信息覆盖程度。

其技术挑战点一般有两点：

1. **查询组合问题**。若组合太多既会对访问网站造成过大压力，也可能会有重复数据的问题。
2. **查询文本框问题**。即有些需要我们填写内容在此网页搜索才能得到内容。

#### 查询组合问题

在查询组合问题中，Google对此提出了较为经典的**富含信息查询模板（Informative Query Templates）技术**。

该技术定义如下：对于某个固定的查询模板来说，如果给模板内每个属性都赋值，形成不同的查询组合，提交给垂直搜索引擎，观察所有返回页面的内容，**如果相互之间内容差异较大，则这个查询模板就是富含信息查询模板**。

为了进一步减少提交的查询数目，Google的技术方案使用了ISIT算法。该算法基本思路为**从一维开始，若一维模板含有富含信息查询模板，若是的话则扩展到二维，如此类推逐步增加维数，直到再也无法找到富含信息查询模板为止**。

> Google的评测结果证明，这种方法和完全组合方式比，能够大幅度提升系统效率。在数据挖掘领域中此算法和数据挖掘里经典的Apriori规则挖掘算法其核心思想相同。

#### 查询文本框问题

在此例中，我们可以通过人工观察网站进行定位，提供一个与网站内容相关的初始种子查询关键词表，以此作为爬虫能够继续工作的基础条件。

爬虫根据初始种子词表，向垂直搜索引擎提交查询，并下载返回的结果页面。之后从返回结果页面里自动挖掘出相关的关键词，并形成一个新的查询列表，依次将新挖掘出的查询提交给搜索引擎。如此往复，直到无法下载到新的内容为止。

**通过这种人工启发结合递归迭代得到种子词表的方式，尽可能覆盖数据库里的记录。**

### 分布式爬虫

采取分布式架构的目的在于提升海量数据抓取的效率。

> 大型分布式爬虫的3个层级：分布式数据中心、分布式抓取服务器及分布式爬虫程序。

对于同一数据中心的多台抓取服务器，不同机器之间的分工协同方式会有差异，常见的分布式架构有两种：

- 主从式分布爬虫
- 对等式分布爬虫

![](https://img.zhengyua.cn/img/20220227110033.png)

> Google在早期即采用此种主从分布式爬虫，在这种架构中，因为URL服务器承担很多管理任务，同时待抓取URL队列数量巨大，所以URL服务器容易成为整个系统的瓶颈。

![](https://img.zhengyua.cn/img/20220227110155.png)

> 为了解决哈希取模的对等式分布爬虫存在的问题，UbiCrawler爬虫提出了改进方案，即放弃哈希取模方式，转而**采用一致性哈希方法（Consisting Hash）来确定服务器的任务分工**。

## 第三章 搜索引擎索引

### 3.1 索引基础

#### 单词-文档矩阵

![](https://img.zhengyua.cn/img/20220227110817.png)

搜索引擎的索引其实就是实现单词—文档矩阵的具体数据结构。可以有不同的方式来实现上述概念模型，比如倒排索引、签名文件、后缀树等方式。其中倒排索引是文档映射关系的最佳实现方式。

#### 倒排索引

![](https://img.zhengyua.cn/img/20220227111559.png)

构建倒排索引时的主要关注点：

- 分词系统
- 索引记载信息

![](https://img.zhengyua.cn/img/20220227112914.png)

### 3.2 单词词典

构造和查询单词词典所使用的数据结构一般有：

- 哈希加链表结构
- 树形词典结构

### 3.3 倒排列表

![](https://img.zhengyua.cn/img/20220227113537.png)

为了更好对数据进行压缩，在实际的搜索引擎系统中，并不存储倒排索引项中的实际文档编号，而是以文档编号差值（D-Gap）。

### 3.4 建立索引

#### 两遍文档遍历法（2-Pass In-Memory Inversion）

此方法完全是在内存里完成索引的创建过程的。

![](https://img.zhengyua.cn/img/20220227113921.png)

- 第一遍扫描的主要目的是**资源准备工作**（如获得统计信息、以此分配内存等资源、建立内存中单词文档位置信息等）；
- 第二遍扫描**真正建立每个单词的倒排列表信息**，填充分配好的内存空间；
- 两遍扫描后将内存的倒排列表和词典信息写入磁盘；

缺点在于：

- 对内存有要求；
- 速度性能较差（主要因为两次遍历中磁盘读取消耗较大）；

#### 排序法（Sort-based Inversion）

该方法在建立索引的过程中，**始终在内存中分配固定大小的空间，用来存放词典信息和索引的中间结果，若内存空间不足则将中间结果排序处理后写入磁盘，同时清理内存空间。最终将磁盘写入的中间结果进行排序合并得到最终的索引。**



![](https://img.zhengyua.cn/img/20220227115007.png)

![](https://img.zhengyua.cn/img/20220227115209.png)

#### 归并法（Merge-based Inversion）

归并法是在排序法的基础上进行优化，改进点主要在将内存中维护的词典信息等所有中间结果也写入磁盘，利用后续内存空间使用。

![](https://img.zhengyua.cn/img/20220227115832.png)

虽然流程与排序法大致相同，但是实现方式差异较大：

- 存放在内存的中间结果，归并法是建议了完整的索引结构（完整倒排索引），而排序法则只是构建的并不完整；
- 中间结果写入磁盘临时文件，归并法是将完整内存倒排索引以词典项在前倒排列表在后的规律写入临时文件，直至内存清空，而排序法并没有将词典映射表写入；

### 3.5 动态索引

![](https://img.zhengyua.cn/img/20220227150146.png)

这种动态索引中，有3个关键的索引结构：

- 倒排索引
- 临时索引
- 已删除文档列表

> 对于更新的处理可以认为是先删后增。

### 3.6 索引更新策略

动态索引通过在内存中维护临时索引，可以实现对动态文档和实时搜索的支持。**但是服内存总是有限的，当内存被消耗完需要考虑将临时索引更新到磁盘索引中，释放内存空间**，此时就涉及到了索引的更新策略。

常见的索引更新策略如下：

- 完全重建策略
- 再合并策略
- 原地更新策略
- 混合策略

#### 完全重建策略（Complete Re-Build）

该策略的基本思路为**直接将新老文档合并后重新建立新索引，完成后丢弃老索引。**

![](https://img.zhengyua.cn/img/20220227151021.png)

比较适合小文档集合，因为完全重建索引的代价较高。

#### 再合并策略（Re-Merge）

此策略就是把**将写入磁盘的临时索引和老文档的倒排索引进行合并生成新的索引。**

![](https://img.zhengyua.cn/img/20220227151244.png)

![](https://img.zhengyua.cn/img/20220227151548.png)

需要说明的是合并时老倒排文件和增量索引中的词典都已排序过，这样可以顺序读取文件内容，减少磁盘寻道时间，这是其高效的根本原因。而该策略缺点在于老索引中很多单词没有变化也需要重新进行写入构建新索引，造成不必要的磁盘消耗。

#### 原地更新策略（In-Place）

该策略出发点就是解决再合并策略中的局限：

- 只对倒排列表变化的单词进行处理，而不需要对未发生变化的老索引进行处理；
- 即使老索引的倒排列表更新直接在末尾进行追加操作，而不需要读取原先的倒排列表并重写到磁盘；

所以该策略的主要优化在于：

- 在索引合并时**直接在原先老的索引文件里进行追加操作**，将增量索引里单词的倒排列表项追加到老索引相应位置的末尾；
- 为了支持上面的追加操作，在初始建立的索引时为**每个单词的倒排列表末尾预留磁盘空间**；

> 若预留空间不足，就需要进行扩容“迁移”即寻找到更大的磁盘空间后将老索引文件重新写入，然后进行追加操作。

![](https://img.zhengyua.cn/img/20220227152518.png)

但是实验数据证明其索引更新效率比再合并策略低，原因如下：

- 扩容“迁移”**操作较为频繁**，且需要对磁盘可用空间进行维护和管理的**成本较高**；
- “迁移”操作破坏了索引文件中的单词连续性 **（降低读取速度）** ，若在索引文件合并时顺序读取则需要维护单词及其倒排文件的映射表 **（映射表维护管理成本较高）**；


#### 混合策略（Hybrid）

顾名思义即希望能够结合不同索引更新策略。

常见的做法如下：

- 出现较为频繁的单词倒排列表较长即长倒排列表单词，这里就采取原地更新策略，解决磁盘读写开销的瓶颈；
- 出现较少的单词倒排列表就较短即短倒排列表单词，这里采取再合并策略，解决顺序读写的瓶颈；

### 3.7 查询处理

目前有两种常见的查询处理机制:

- 一次一文档方式;
- 一次一单词方式；
- 跳跃指针方式（优化）；

#### 一次一文档（Doc at a Time）

以倒排列表中包含的文档为单位，**每次将其中某个文档与查询的最终相似性得分计算完毕**，然后开始计算另外一个文档的最终得分，直到所有文档的得分都计算完毕为止。

> 搜索系统的输出结果往往是限定个数的，在实际实现一次一文档方式时，不必保存所有文档的相关性得分，而只需要在内存中维护一个大小为K的优先级别队列（根堆数据结构）。

![](https://img.zhengyua.cn/img/20220227154612.png)

#### 一次一单词（Term at a Time）

![](https://img.zhengyua.cn/img/20220227155518.png)

一次一文档可以直观理解为在单词—文档矩阵中，以文档为单位，纵向进行分数累计，之后移动到后续文档接着计算，即计算过程是“先纵向再横向”；**而一次一单词则是采取“先横向再纵向”的方式**。

#### 跳跃指针（Skip Pointers）

如果用户输入的查询包含多个查询词，搜索引擎一般默认是采取**与逻辑**来判别文档是否满足要求。

对于多词查询，找到包含所有查询词的文档，等价于求查询词对应的倒排列表的交集。（一次一文档）但在求其交集时，**若倒排列表的文档ID以文档编号差值（D-Gap）形式存储，且差值是以压缩后的方式进行编码的，则会变得复杂**（先读入内存后进行压缩恢复到差值形式，然后恢复到有序列表，最后进行交集运算）。而跳跃指针就是为了优化该计算过程。

其基本思想是将一个倒排列表数据化整为零，切分为若干个固定大小的数据块，一个数据块作为一组，对于每个数据块，增加元信息来记录关于这个块的一些信息，这样即使是面对压缩后的倒排列表的优势也有：

- 只解压缩部分数据即可；
- 无须比较任意两个文档ID；

> 在实际应用中，如何设定数据块或者数据组的大小对于效率有影响，实践表明一个简单有效的启发规则是：假设倒排列表长度为L（即包含L个文档ID），使用L作为块大小，则效果较好。

![](https://img.zhengyua.cn/img/20220227160602.png)

### 3.8 多字段索引

**区分不同字段对于搜索引擎的相关性评分也有很大帮助。** 

搜索引擎需要能够对多字段进行索引，去其实现多字段索引常见方式有以下：

- 多索引方式；
- 倒排列表方式；
- 扩展列表方式；

#### 多索引方式

当用户没有指定特定字段时，**搜索引擎会对所有字段都进行查找并合并多个字段的相关性得分**，对于多索引方式来说，就需要对多个索引进行读取，所以这种方式的效率会比较低。

![](https://img.zhengyua.cn/img/20220301154024.png)

#### 倒排列表方式

**将字段信息存储在某个关键词对应的倒排列表内**，实现在读到关键词时可以根据倒排列表中的字段信息直接进行过滤和并保留指定字段内出现过搜索词的文档作为搜索结果返回。

![](https://img.zhengyua.cn/img/20220301154509.png)

#### 扩展列表方式

**为每个字段建立一个列表，记载每个文档这个字段对应的出现位置信息**。

![](https://img.zhengyua.cn/img/20220301155030.png)

### 3.9 短语查询

若单词的倒排列表只存储文档编号和单词频率信息，其保留的信息是不足以支持短语搜索的，因为**单词之间的顺序关系没有保留**。

搜索引擎支持短语查询，**本质问题是如何在索引中维护单词之间的顺序关系或者位置信息**。较常见的支持短语查询的技术方法包括：

- 位置信息索引
- 双词索引
- 短语索引

#### 位置信息索引（Position Index）

使用**索引记载单词位置信息**来支持短语查询，但效率较低和成本较高。

![](https://img.zhengyua.cn/img/20220301155654.png)

#### 双词索引（Nextword Index）

由于维度增加，**双词索引使得索引和倒排列表指数级增长**，所以一般对于计算和存储代价高使用

![](https://img.zhengyua.cn/img/20220301160039.png)

#### 短语索引（Phrase Index）

短语索引就是在词典中**直接加入多词短语并维护短语的倒排列表**以此对短语进行支持。

局限性在于需要手动挖掘热门短语，对于其他短语查询采用普通方式处理。

![](https://img.zhengyua.cn/img/20220301160814.png)

#### 混合索引

混合索引即结合上述索引的优缺点根据具体应用场景进行设计。

![](https://img.zhengyua.cn/img/20220301160956.png)


### 3.10 分布式索引

多台机器如何分工协作，目前常用的分布式索引方案包括两种：

- 按文档对索引划分
- 按单词对索引划分

#### 按文档划分（Document Partitioning）

将整个文档集合切割成若干子集合，而每台机器负责对某个文档子集合建立索引，并响应查询请求。

主要过程就是查询分发服务器综合各个索引服务器的搜索结果后，合并搜索结果，将得分最高的文档作为最终搜索结果返回给用户。

![](https://img.zhengyua.cn/img/20220301161553.png)

#### 按单词划分（Term Partitioning）

每个索引服务器负责词典中部分单词的倒排列表的建立和维护。

![](https://img.zhengyua.cn/img/20220301161850.png)

#### 比较

以上两种分布式索引技术方案，**按文档来对索引进行划分是比较常用的**，而按单词进行索引划分只在比较特殊的应用场合才使用。相较于文档对索引进行划分，后者有以下局限性：

- 可扩展性（会影响到所有索引服务器）
- 负载均衡（如热点索引场景）
- 容错性（若索引服务器宕机则导致直接查询失败）
- 对查询方式处理（只支持一次一单词处理方式）