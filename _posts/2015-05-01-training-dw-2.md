---
layout: post
category: "数据仓库"
title: "阿里云[2]－ 基于ODPS+SLS+DPC搭建数据平台"
tags: ["数据仓库"]
---

#### 1.名词解释：
- ODPS:开放数据处理服务（Open Data Processing Service）是基于飞天分布式平台，由阿里云自主研发的海量数据离线处理服务。ODPS以RESTful API的形式提供针对PB级别数据的、实时性要求不高的批量结构化数据存储和计算能
- DPC:采云间（Data Process Center）是基于开放数据处理服务（ODPS）的DW/BI的工具解决方案。DPC提供全链路的易于上手的数据处理工具，包括ODPS IDE、任务调度、数据分析、报表制作和元数据管理等
- SLS:简单日志服务（Simple Log Service，简称SLS）是针对日志收集、存储、查询和分析的服务。SLS可收集云服务和应用程序生成的日志数据并编制索引，提供实时查询海量日志的能力
- 数据仓库:数据仓库之父比尔·恩门（Bill Inmon）在1991年出版的《建立数据仓库》一书中所提出的定义被广泛接受，数据仓库是一个面向主题的、集成的、相对稳定的、反映历史变化的数据集合，用于支持管理决策。 数据仓库是一个过程而不是一个项目；数据仓库是一个环境，而不是一件产品。数据仓库提供用户用于决策支持的当前和历史数据，这些数据在传统的操作型数据库中很难或不能得到。数据仓库技术是为了有效的把操作形数据集成到统一的环境中以提供决策型数据访问，的各种技术和模块的总称。所做的一切都是为了让用户更快更方便查询所需要的信息，提供决策支持。


#### 2.数据流程：
![Alt text](/images/product-relation.png)
<br>
![Alt text](/images/product-marts.jpg)
<br>
![Alt text](/images/dw.jpg)

>
- 请尊重本人劳动成功，可以随意转载但保留以下信息 
- 作者：岁月经年 
- 时间：2015年4月
