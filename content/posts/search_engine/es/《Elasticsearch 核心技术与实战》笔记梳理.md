---
title: "《Elasticsearch 核心技术与实战》笔记梳理"
date: 2022-03-14T15:00:00+08:00
draft: true
tags: ["search_engine","note","es"]
categories: ["search engine"]
slug: /search_engine/es/8
---

- **作者**：阮一鸣 eBay Pronto 平台技术负责人
- **課程地址**：https://time.geekbang.org/course/intro/100030501?tab=catalog


第一章：概述 (4讲)
01 | 课程介绍
02 | 内容综述及学习建议
03 | Elasticsearch简介及其发展历史
04 | Elastic Stack家族成员及其应用场景

[ElasticSearch初识](https://zhengyua.cn/posts/search_engine/es/0.html/)

第二章：安装上手 (4讲)
05 | Elasticsearch的安装与简单配置
06 | Kibana的安装与界面快速浏览
07 | 在Docker容器中运行Elasticsearch Kibana和Cerebro
08 | Logstash安装与导入数据

[ElasticSearch初识](https://zhengyua.cn/posts/search_engine/es/0.html/)

第三章：Elasticsearch入门 (15讲)
09 | 基本概念：索引、文档和REST API
10 | 基本概念：节点、集群、分片及副本
11 | 文档的基本CRUD与批量操作

[ElasticSearch基本概念](https://zhengyua.cn/posts/search_engine/es/1.html/)

12 | 倒排索引介绍
13 | 通过Analyzer进行分词

[ElasticSearch倒排索引&Analysis分词](https://zhengyua.cn/posts/search_engine/es/2.html/)

14 | Search API概览
15 | URI Search详解
16 | Request Body与Query DSL简介
17 | Query String&Simple Query String查询 

[ElasticSearchSearchAPI](https://zhengyua.cn/posts/search_engine/es/3.html/)

18 | Dynamic Mapping和常见字段类型
19 | 显式Mapping设置与常见参数介绍

[ElasticSearchMapping](https://zhengyua.cn/posts/search_engine/es/4.html/)

20 | 多字段特性及Mapping中配置自定义Analyzer

[ElasticSearch多字段特性&自定义Analyzer](https://zhengyua.cn/posts/search_engine/es/5.html/)

21 | Index Template和Dynamic Template

[ElasticSearchTemplate使用](https://zhengyua.cn/posts/search_engine/es/6.html/)

22 | Elasticsearch聚合分析简介

[ElasticSearch聚合简介](https://zhengyua.cn/posts/search_engine/es/7.html/)

23 | 第一部分总结

第四章：深入搜索 (13讲)
24 | 基于词项和基于全文的搜索
25 | 结构化搜索
26 | 搜索的相关性算分
27 | Query&Filtering与多字符串多字段查询
28 | 单字符串多字段查询：Dis Max Query
29 | 单字符串多字段查询：Multi Match
30 | 多语言及中文分词与检索
31 | Space Jam，一次全文搜索的实例
32 | 使用Search Template和Index Alias查询
33 | 综合排序：Function Score Query优化算分
34 | Term&Phrase Suggester
35 | 自动补全与基于上下文的提示
36 | 配置跨集群搜索
第五章：分布式特性及分布式搜索的机制 (8讲)
37 | 集群分布式模型及选主与脑裂问题
38 | 分片与集群的故障转移
39 | 文档分布式存储
40 | 分片及其生命周期
41 | 剖析分布式查询及相关性算分
42 | 排序及Doc Values&Fielddata
43 | 分页与遍历：From, Size, Search After & Scroll API
44 | 处理并发读写操作
第六章：深入聚合分析 (4讲)
45 | Bucket & Metric聚合分析及嵌套聚合
46 | Pipeline聚合分析
47 | 作用范围与排序
48 | 聚合分析的原理及精准度问题
第七章：数据建模 (7讲)
49 | 对象及Nested对象
50 | 文档的父子关系
51 | Update By Query & Reindex API
52 | Ingest Pipeline & Painless Script
53 | Elasticsearch数据建模实例
54 | Elasticsearch数据建模最佳实践
55 | 第二部分总结回顾
第八章：保护你的数据 (3讲)
56 | 集群身份认证与用户鉴权
57 | 集群内部安全通信
58 | 集群与外部间的安全通信
第九章：水平扩展Elasticsearch集群 (6讲)
59 | 常见的集群部署方式
60 | Hot & Warm架构与Shard Filtering
61 | 分片设计及管理
62 | 如何对集群进行容量规划
63 | 在私有云上管理Elasticsearch集群的一些方法
64 | 在公有云上管理与部署Elasticsearch集群
第十章：生产环境中的集群运维 (10讲)
65 | 生产环境常用配置与上线清单
66 | 监控Elasticsearch集群
67 | 诊断集群的潜在问题
68 | 解决集群Yellow与Red的问题
69 | 提升集群写性能
70 | 提升集群读性能
71 | 集群压力测试
72 | 段合并优化及注意事项
73 | 缓存及使用Breaker限制内存使用
74 | 一些运维的相关建议
第十一章：索引生命周期管理 (2讲)
75 | 使用Shrink与Rollover API有效管理时间序列索引
76 | 索引全生命周期管理及工具介绍
第十二章：用Logstash和Beats构建数据管道 (3讲)
77 | Logstash入门及架构介绍
78 | 利用JDBC插件导入数据到Elasticsearch
79 | Beats介绍
第十三章：用Kibana进行数据可视化分析 (4讲)
80 | 使用Index Pattern配置数据
81 | 使用Kibana Discover探索数据
82 | 基本可视化组件介绍
83 | 构建Dashboard
第十四章：探索X-Pack套件 (6讲)
84 | 用Monitoring和Alerting监控Elasticsearch集群
85 | 用APM进行程序性能监控
86 | 用机器学习实现时序数据的异常检测（上）
87 | 用机器学习实现时序数据的异常检测（下）
88 | 用ELK进行日志管理
89 | 用Canvas做数据演示
实战1：电影搜索服务 (3讲)
90 | 项目需求分析及架构设计
91 | 将电影数据导入Elasticsearch
92 | 搭建你的电影搜索服务
实战2：Stackoverflow用户调查问卷分析 (3讲)
93 | 需求分析及架构设计
94 | 数据Extract & Enrichment
95 | 构建Insights Dashboard
备战：Elastic认证 (5讲)
96 | Elastic认证介绍
97 | 考点梳理
98 | 集群数据备份
99 | 基于Java和Elasticseach构建应用
100 | 结课测试&结束语