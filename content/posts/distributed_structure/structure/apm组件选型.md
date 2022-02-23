---
title: "apm组件选型"
date: 2022-01-29T17:00:06+08:00
draft: false
tags: ["分布式架构","apm"]
categories: ["分布式架构"]
slug: /distributed_structure/structure/8
---

# APM组件选型

1. 探针的性能

   重点在agent对服务的吞吐量、CPU和内存的影响。

   微服务的规模和动态性使数据收集的成本提高。

2. collector的可扩展性

   水平扩展以便支持更大规模服务器集群。

3. 全面的调用链路数据分析

   提供代码级别的可见性，定位失败点和瓶颈。

4. 对于开发透明，容易开关

   无需修改代码添加新功能，容易启用或者禁用。

5. 完整的调用链应用拓扑

   自动检测应用拓扑，应用架构。

![image-20201204000034061](https://img.zhengyua.cn/img/20201204000054.png)

## 参考

https://juejin.im/post/5a7a9e0af265da4e914b46f1