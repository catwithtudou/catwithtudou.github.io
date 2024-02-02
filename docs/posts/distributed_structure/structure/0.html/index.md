# 分布式系统架构的本质note


# 分布式系统架构的本质note

> 以下为[左耳听风](https://time.geekbang.org/column/intro/48) 记录的笔记。

## 为什么需要分布式系统

放弃传统的单体架构，选择分布式系统，主要有两方面原因：

- 增大系统容量。

  当业务量越来越大，一台机器的性能无法满足，需要多台机器才能应对大规模应用场景，所以需要垂直或者水平拆分业务系统，使其变成分布式架构。

- 加强系统可用。

  业务越来越关键，需要提高整个系统架构的可用性，意味着架构中不能存在单点故障，为避免一台机器故障导致整体不可用，所以需要通过分布式架构来冗余系统以消除单点故障。

> 还有其他优势：
>
> - 模块化，系统模块重用度高；
> - 软件服务模块被拆分，开发和发布速度可以并行而变得更快；
> - 系统扩展性更高；
> - 团队协作流程得到改善等。

![image-20201231201827935](https://img.zhengyua.cn/img/20201231201835.png)

## 分布式系统的发展

开发、维护和使用 SOA 要遵循以下几条基本原则：

- 可重用，粒度合适，模块化，可组合，构件化以及有互操作性；
- 符合开放标准（通用的或行业的）；
- 服务的识别和分类，提供和发布，监控和跟踪。

SOA架构的演化图：

![image-20201231202201323](https://img.zhengyua.cn/img/20201231202201.png)

- 单体架构，软件模块高度耦合；
- 通过标准的协议或是中间件来联动其它相关联的服务来实现松耦合，利用IOC（控制反转）和DIP（依赖倒置原则）设计思想进行实践；
- 业务拆分成多个微服务，微服务独立完整运行，服务间的整合需要服务编排或是服务整合的引擎，可以使工作流引擎，也可以是网关，也需要辅助于像容器化调度的技术方式来进行管理。

微服务的出现使得开发速度变得更快，部署快，隔离性高，系统的扩展度也很好，但是在集成测试、运维和服务管理等方面难度提升，所以需要一套较好的微服务PaaS平台。

## 分布式系统中需要注意的问题

### 异构系统的不标准问题

主要表现在：

- 软件和应用不标准；
- 通讯协议不标准；
- 数据格式不标准；
- 开发和运维的过程和方法不标准；

### 系统架构中的服务依赖性问题

服务依赖会导致的问题主要有以下场景：

- 如果非关键业务被关键业务所依赖，会导致非关键业务变成一个关键业务。
- 服务依赖链中，出现“木桶短板效应”——整个 SLA 由最差的那个服务所决定。

这就涉及到服务治理的内容。

> 不但要拆分服务，还要为每个服务拆分相应的数据库。

### 故障发生的概率更大

- 出现故障不可怕，故障恢复时间过长才可怕。
- 出现故障不可怕，故障影响面过大才可怕。

所谓 Design for Failure。在设计时就要考虑如何减轻故障。如果无法避免，也要使用自动化的方式恢复故障，减少故障影响面。

### 多层架构的运维复杂度更大

通常来说可以把系统分成四层：基础层、平台层、应用层和接入层。

- 基础层：机器、网络和存储设备等。
- 平台层：中间件层，Tomcat、MySQL、Redis、Kafka 之类的软件。
- 应用层：业务软件，比如，各种功能的服务。
- 接入层：接入用户请求的网关、负载均衡或是 CDN、DNS 类似。

需要注意：

- 任何一层的问题都会导致整体的问题；
- 没有统一的视图和管理，导致运维被割裂开来，造成更大的复杂度。

## 分布式系统的技术栈

> 构建分布式系统的目的是为了增加系统容量，提高系统的可用性，转换为技术方面，就是以下两件事情，即**一是提高整体架构的吞吐量，服务更多的并发和流量，二是为了提高系统的稳定性，让系统的可用性更高。**
>
> - 大流量处理。
> - 关键业务保护。

### 提高架构的性能

![image-20210101115444787](https://img.zhengyua.cn/img/20210101115451.png)

- 缓存系统

  加入缓存系统，可以有效地提高系统的访问能力。对于分布式系统下的缓存系统，需要的是一个缓存集群。这其中需要一个 Proxy 来做缓存的分片和路由。

- 负载均衡系统

  水平扩展的关键技术，使用多台机器来共同分担一部分流量请求。

- 异步调用

  主要通过消息队列来对请求做排队处理，来达到削峰填谷的目的，但是会带来实时性的问题，同时需要通过消息持久化来解决消息丢失的问题，但这会造成“有状态”的结点，增加了服务调度的难度。

- 数据分区

  把数据按一定的方式分成多个区，不同的数据区来分担不同区的流量。这需要一个数据路由的中间件，会导致跨库的 Join 和跨库的事务非常复杂。

- 数据镜像

  把一个数据库镜像成多份一样的数据，这样就不需要数据路由的中间件了。为了达到内部同步的读写一致，数据镜像中最大的问题就是数据的一致性问题。

### 提高架构的稳定性

![image-20210101115935777](https://img.zhengyua.cn/img/20210101115935.png)

- 服务拆分

  主要有两个目的：一是为了隔离故障，二是为了重用服务模块。但服务拆分完之后，会引入服务调用间的依赖问题。

- 服务冗余

  服务冗余是为了去除单点故障，并可以支持服务的弹性伸缩，以及故障迁移。但是对于某些有状态的服务来说，冗余这些有状态的服务带来了更高的复杂性。其中一个是弹性伸缩时，需要考虑数据的复制或是重新分片，迁移的时候还要迁移数据到其它机器上。

- 限流降级

  通过限流或者功能降级的方式来停掉一部分服务，或是拒绝一部分用户，以确保整个架构不会挂掉。这些技术属于保护措施。

- 高可用架构

  从冗余架构的角度来保障可用性，如多租户隔离，灾备多活，或是数据可以在其中复制保持一致性的集群。主要目的就是解决单点故障。

- 高可用运维

  指的是 DevOps 中的 CI/CD（持续集成 / 持续部署）。良好的运维应该是一条很流畅的软件发布管线，其中涉及到自动化测试和相应的灰度发布，以及对线上系统的自动化控制。这样，可以做到“计划内”或是“非计划内”的宕机事件的时长最短。

### 分布式系统的关键技术

- 服务治理

  服务拆分、服务调用、服务发现、服务依赖、服务的关键度定义等，服务治理的最大意义是梳理服务间的依赖关系、服务调用链，以及关键的服务，且对这些服务进行性能和可用性方面的管理。

- 架构软件管理

  因为服务之间有依赖且出现兼容性问题，所以整体服务所形成的架构需要有架构版本管理、整体架构的生命周期管理，以及对服务的编排、聚合、事务处理等服务调度功能。

- DevOps

  为了分布式系统可以更为快速地更新服务，且对于服务的测试和部署，所以需要 DevOps 的全流程，其中包括环境构建、持续集成、持续部署等。

- 自动化运维

  通过DevOps对服务进行自动伸缩、故障迁移、配置管理、状态管理等一系列的自动化运维技术。

- 资源调度管理

  应用层的自动化运维需要基础层的调度支持，即云计算 IaaS 层的计算、存储、网络等资源调度、隔离和管理。

- 整体架构监控

  监控需要对三层系统（应用层、中间件层、基础层）进行监控。

- 流量控制

  包括负载均衡、服务路由、熔断、降级、限流等和流量相关的调度，且包括灰度发布之类的功能等。

### 分布式系统纲领

分布式系统关键技术总结下来有以下：

- 全栈系统监控；
- 服务 / 资源调度；
- 流量调度；
- 状态 / 数据调度；
- 开发和运维的自动化。

![image-20210101120705056](https://img.zhengyua.cn/img/20210101120705.png)

## 分布式系统关键技术：监控

> 监控系统需要完成的功能：
>
> - 全栈监控；
> - 关键分析；
> - 跨系统调用的串联；
> - 实时报警和自动处置；
> - 系统性能分析；

### 全栈监控

全栈监控简单来说就是三层监控：

- **基础层**：监控主机和底层资源。

  比如：CPU、内存、网络吞吐、硬盘 I/O、硬盘使用等。

- **中间层**：就是中间件层的监控。

  比如：Nginx、Redis、ActiveMQ、Kafka、MySQL、Tomcat 等。

- **应用层**：监控应用层的使用。

  比如：HTTP 访问的吞吐量、响应时间、返回码、调用链路分析、性能瓶颈，还包括用户端的监控。

![image-20210102233301038](https://img.zhengyua.cn/img/20210102233308.png)

监控的标准化：

- 日志数据结构化；
- 监控数据格式标准化；
- 统一的监控平台；
- 统一的日志分析；

### 好的监控

监控系统主要存在两个很大的问题：

- 监控数据是隔离开来的。
- 监控的数据项太多。

一个好的监控系统应该有以下几个特征：

- **关注于整体应用的SLA**。主要从为用户服务的 API 来监控整个系统。
- **关联指标聚合**。把有关联的系统及其指标聚合展示。主要是三层系统数据：基础层、平台中间件层和应用层。把服务和相关的中间件以及主机关联在一起，尤其需要把服务的具体实例和主机关联在一起。
- **快速故障定位**。以跟踪一个用户请求的trace来监控出现整个分布式系统上的调用链，最好是做成没有侵入性的。

一个好的监控系统主要是为以下两个场景设计的：

1. 体验

- **容量管理**。提供全局系统运行时数据展示，可以判断是否需要增加机器或者其他资源。
- **性能管理**。通过查看Dashborad找到系统瓶颈，针对性地优化系统和相应代码。

2. 急诊

- **定位问题**。快速暴露并找到问题的发生点，以助于诊断问题。
- **性能分析**。当出现非预期的流量提升时，可以快速找到系统瓶颈，帮助开发人员深入代码。

### 如何实现好的监控系统

- 服务调用链跟踪

  对外API——关联后台实际服务——服务依赖关联——直至最后一个服务。

  ![image-20210102234348688](https://img.zhengyua.cn/img/20210102234348.png)

- 服务调用时长分布

  服务调用链上的时间分布。

  ![image-20210102234419023](https://img.zhengyua.cn/img/20210102234419.png)

- 服务的TOPN视图

  系统请求的排名情况，可根据调用量、请求耗时、热点进行排名。

  ![image-20210102234451055](https://img.zhengyua.cn/img/20210102234451.png)

- 数据库关联操作

  执行数据库操作的执行时间，将相关请求对应起来。

  ![image-20210102234526041](https://img.zhengyua.cn/img/20210102234526.png)

- 服务资源跟踪

  将服务运行的机器节点上的数据关联。

在把数据收集好的同时，更重要的是把数据关联好，才能够很快地定位故障，进而才能进行自动化调度。如下图所出现的情况。

![image-20210102234654207](https://img.zhengyua.cn/img/20210102234654.png)

## 分布式系统关键技术：服务调度

> 关键技术，主要有以下几点：
>
> - 服务关键程度；
> - 服务依赖关系；
> - 服务发现；
> - 整个架构的版本管理；
> - 服务应用声明周期安全管理；

### 服务关键程度和服务的依赖关系

- 服务关键程度，主要是要我们梳理和定义服务的重要程度。
- 服务间的依赖关系，传统的SOA希望通过ESB来解决服务间的依赖关系，而微服务时服务依赖最优解的上限，而下限是千万不要有依赖环。
- 解决服务依赖环的方案一般是依赖倒置的设计模式，即在分布式架构上通过一个第三方的服务来解决这个事。
- 服务依赖关系可通过技术手段发现，如Zipkin服务调用跟踪系统。

### 服务状态和生命周期的管理

- 需要服务发现的中间件来管理架构的服务、服务版本、服务实例、服务状态等。
- 有了服务的状态和运行情况后，就需要对服务的生命周期进行管理，服务的生命周期通常有以下几个状态：
    - Provision，代表在供应一个新的服务；
    - Ready，表示启动成功了；
    - Run，表示通过了服务健康检查；
    - Update，表示在升级中；
    - Rollback，表示在回滚中；
    - Scale，表示正在伸缩中（可以有 Scale-in 和 Scale-out 两种）；
    - Destroy，表示在销毁中；
    - Failed，表示失败状态。

### 整个架构的版本管理

- VersionSet，由一堆服务的版本集所形成的整个架构的版本控制。
- 考虑到版本兼容性的问题，除了各个项目的版本管理之外，还需要在上面再盖一层版本管理。
- 版本管理为实现服务之间的版本兼容性，则需要定义服务清单即定义了所有服务的版本运行环境，其中包括但不限于：
    - 服务的软件版本；
    - 服务的运行环境——环境变量、CPU、内存等；
    - 服务运行的最大最小实例数；

### 资源/服务调度

服务和资源调度的过程，与操作系统调度进程的方式很相似，主要有以下一些关键技术：

- 服务状态的维持和拟合；

  服务的运行状态，包括一些非预期的变化和预期的变化。

  > 在发布新版本时，需要伸缩，需要回滚。集群管理控制器会把集群从现有的状态迁移到另一个状态，它需要一步一步地向集群发送若干控制命令，即这个过程为“拟合”。
  >
  > 比如，当需要对集群进行 Scale 的时候，我们需要：
  >
  > - 先扩展出几个结点；
  > - 再往上部署服务；
  > - 然后启动服务；
  > - 再检查服务的健康情况；
  > - 最后把新扩展出来的服务实例加入服务发现中提供服务。

- 服务的弹性伸缩和故障迁移；

  服务伸缩所需要的操作步骤，其中涉及到了：

    - 底层资源的伸缩；
    - 服务的自动化部署；
    - 服务的健康检查；
    - 服务发现的注册；
    - 服务流量的调度。

  而对于故障迁移，也就是服务的某个实例出现问题时则需要自动地恢复它。对于服务来说，有两种模式，一种是宠物模式，一种是奶牛模式。

  对于这两种模式，在运行中也是比较复杂的，其中涉及到了：

    - 服务的健康监控（这可能需要一个 APM 的监控）。
    - 如果是宠物模式，需要：服务的重新启动和服务的监控报警（如果重试恢复不成功，需要人工介入）。
    - 如果是奶牛模式，需要：服务的资源申请，服务的自动化部署，服务发现的注册，以及服务的流量调度。

- 作业和应用调度；

- 作业工作流编排；

- 服务编排；

  一个好的操作系统需要能够通过一定的机制把一堆独立工作的进程给协同起来。在分布式的服务调度中，这个工作叫做 Orchestration，即“编排”。

  > 注意，ESB 的服务编制叫 Choreography，与Orchestration是不一样的，前者较为重量级。
  >
  > - 后者的意思是所有服务的交互是通过一个总控制者来进行编排。
  > - 前者的意思是在各自完成专属自己的工作的基础上考虑怎样互相协作。

  一般来说，可以通过API Gateway或者一个简单的消息队列来做相应的编排工作，如Zuul、Ingress相似等。

  关于服务的编排会直接导致一个服务编排的工作流引擎中间件的产生。

## 分布式系统关键技术：流量和数据调度

> 流量调度与服务治理需要区分开来，它们属于不同层面：
>
> - 服务治理是内部系统的事，而流量调度可以是内部的，更是外部接入层的事。
> - 服务治理是数据中心的事，而流量调度的最优应该是数据中心之外的事即边缘计算，在类似于 CDN 上完成的事。

#### 流量调度的主要功能

对于流量调度系统来说应具有的主要功能：

- 依据系统运行的情况，**自动地进行流量调度**，在无需人工干预的情况下，提升整个系统的稳定性；
- 让系统应对爆品等突发事件时，**在弹性计算扩缩容的较长时间窗口内或底层资源消耗殆尽的情况下**，保护系统平稳运行。

流量调度系统还可以完成以下几方面的事情：

- **服务流控**。服务发现、服务路由、服务降级、服务熔断、服务保护等。
- **流量控制**。负载均衡、流量分配、流量控制、异地灾备（多活）等。
- **流量管理**。协议转换、请求校验、数据缓存、数据计算等。

> 所有这些都是一个API GateWay应该做的事情。

#### 流量调度的关键技术

调度流量首先需要抗住流量，而且需要一些比较轻量的业务逻辑。

一个好的API GateWay应该具备以下关键技术：

- **高性能**。高性能的技术，也需要使用高性能的语言。
- **抗流量**。关键点在于集群技术，而该技术的关键点为结点之间的数据共享。且因部署在广域网上也需要集群的分组技术。
- **业务逻辑**。需要有简单的业务逻辑。
- **服务化**。能够通过 Admin API 来不停机地管理配置变更。

#### 状态数据调度

- 状态服务会保存数据，且不能丢失，需要随服务一起调度。

一般通过转移到第三方服务使其变为无状态的服务，但数据存储结点在Scale上比较困难，所以成了一个单点的瓶颈。

#### 分布式事务一致性问题

数据 replication 则会带来数据一致性的问题，进而对性能带来严重的影响。

解决数据不丢失或者说保证数据高可用性的问题，本质就是通过**数据冗余**来解决。

解决数据副本间的一致性问题，一般有以下方案：

- M-S方案
- M-M方案
- 两阶段和三阶段提交方案
- Paxos方案

**通过业务补偿的方式**解决数据一致性问题，即是在**应用层**上解决事务问题。

若要在**数据层**解决事务问题，则Paxos算法则可以实现。

#### 数据结点的分布式方案

真正完整解决数据 Scale 问题的应该还是**数据结点自身**。

只有数据结点自身解决了这个问题才能做到对上层业务层的透明，业务层可以像操作单机数据库一样来操作分布式数据库，这样才能做到整个分布式服务架构的调度。即该问题应该解决在**数据存储方**。数据存储结果有太多不同的Scheme，如AWS的Aurora、PingCAP的TiDB、MySQLCluster、阿里的OceanBase等。而对于一些需要文件存储的，则需要**分布式文件系统**的支持，

> 状态数据调度应该是在 IaaS 层的数据存储解决的问题，而不是在 PaaS 层或者 SaaS 层来解决的。

## PaaS平台

### 软件工程能力体现

- **提高服务的SLA**

  主要表现在两个方面：

    - 高可用的系统
    - 自动化的运维

- **能力和资源重用或复用**

  其主要表现在两个方面：

    - 软件模块的重用
    - 软件运行环境和资源的重用

  “软件抽象的能力”和“软件标准化的能力”

- **过程的自动化**

    - 软件生产流水线
    - 软件运维自动化

软件工程的三个本质与分布式技术点是高度一致的，即下面三个方面的能力：

- 分布式多层的系统架构
- 服务化的能力供应
- 自动化的运维能力

### PaaS平台的本质

一个好的PaaS平台应该具有分布式、服务化、自动化部署、高可用、敏捷以及分层开放的特征，并可与IaaS实现良好的联动。

![image-20210106233721178](https://img.zhengyua.cn/img/20210106233728.png)

PaaS和传统中间件最大的差别：

- **服务化是PaaS的本质**

  软件模块重用，服务治理，对外提供能力是PaaS的本质。

- **分布式是PaaS的根本特性**

  多租户隔离、高可用、服务编排是PaaS的基本特性。

- **自动化是PaaS的灵魂**

  自动化部署安装运维，自动化伸缩调度是PaaS的关键。

### PaaS平台的总体架构

![image-20210106234028195](https://img.zhengyua.cn/img/20210106234028.png)

一个完整的PaaS平台会包括以下几个部分：

- **PaaS调度层** – 主要是PaaS的自动化和分布式对于高可用高性能的管理。
- **PaaS能力服务层** – 主要是PaaS真正提供给用户的服务和能力。
- **PaaS的流量调度** – 主要是与流量调度相关的东西，包括对高并发的管理。
- **PaaS的运营管理** – 软件资源库、软件接入、认证和开放平台门户。
- **PaaS的运维管理** – 主要是DevOps相关的东西。

### PaaS平台的生产和运维

![image-20210106234143856](https://img.zhengyua.cn/img/20210106234144.png)

## 小结

为了构建分布式系统，我们面临的主要问题如下：

- 分布式系统的**硬件故障发生率更高**，故障发生是常态，需要尽可能地将**运维流程自动化**。
- 需要良好地设计服务，避免某服务的**单点故障**对依赖它的其他服务造成大面积影响。
- 为了**容量的可伸缩性**，服务的拆分、自治和无状态变得更加重要，可能需要对老的软件逻辑做大的修改。
- 老的服务可能是异构的，此时需要让它们使用**标准的协议**，以便可以被**调度、编排**，且互相之间可以通信。
- 服务软件故障的处理也变得复杂，需要**优化的流程**，以**加快故障的恢复**。
- 为了管理各个服务的容量，让分布式系统发挥出最佳性能，需要有**流量调度技术**。
- 分布式存储会让**事务处理变得复杂**；在事务遇到故障无法被自动恢复的情况下，手动恢复流程也会变得复杂。
- **测试和查错**的复杂度增大。
- 系统的**吞吐量会变大**，但**响应时间会变长**。

为了解决这些问题，我们深入了解了以下这些解决方案：

- 需要有完善的**监控系统**，以便对服务运行状态有全面的了解。
- 设计服务时要**分析其依赖链**；当非关键服务故障时，其他服务要自动**降级功能**，避免调用该服务。
- 重构老的软件，使其能被**服务化**；可以参考SOA和微服务的设计方式，目标是**微服务化**；使用Docker和Kubernetes来**调度服务**。
- 为老的服务编写接口逻辑来使用**标准协议**，或在必要时重构老的服务以使得它们有这些功能。
- 自动构建**服务的依赖地图**，并引入好的**处理流程**，让团队能以最快速度定位和恢复故障。
- 使用一个**API Gateway**，它具备**服务流向控制、流量控制和管理**的功能。
- 事务处理建议在**存储层实现**；根据业务需求，或者降级使用更简单、吞吐量更大的**最终一致性**方案，或者通过二阶段提交、Paxos、Raft、NWR等方案之一，使用吞吐量小的**强一致**方案。
- 通过更真实地**模拟生产环境**，乃至在生产环境中做灰度发布，从而增加测试强度；同时做充分的**单元测试和集成测试**以发现和消除缺陷；最后，在服务故障发生时，相关的多个团队同 时**上线自查服务状态**，以最快地定位故障原因。
- 通过**异步调用**来减少对短响应时间的依赖；对**关键服务**提供专属硬件资源，并优化软件逻辑以缩短响应时间。