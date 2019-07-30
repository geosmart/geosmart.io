---
title: Hbase背景

date: 2017-07-03 20:41:50

tags: [Hbase]

categories: 大数据

---

介绍Hbase出场背景、解决问题、应用场景

<!-- more --> 

# 官方Hbase简介

> Use Apache HBase™ when you need random, realtime read/write access to your Big Data. 

当需对大数据进行`随机`，`实时读/写`时采用Hbase；

> This project's goal is the hosting of very large tables -- billions of rows X millions of columns -- atop clusters of commodity hardware. 

Hbase项目的目标是在集群中存储`数十亿行`，`数百万列`的超级大表；

> Apache HBase is an open-source, distributed, versioned, **non-relational database** modeled after Google's [Bigtable: A Distributed Storage System for Structured Data](https://research.google.com/archive/bigtable.html) by Chang et al. 

Hbase是一个开源的分布式的`非关系型`数据库，其数据模型是基于Google的Bigtable论文（结构化数据的分布式存储系统）开发的；

> Just as Bigtable leverages the distributed data storage provided by the Google File System, Apache HBase provides **Bigtable-like capabilities** on top of Hadoop and HDFS.

正如Google的Bigtable用Google文件系统分布式存储数据，Hbase基于Hadoop的`HDFS(Hadoop File System)`来存储海量数据；

> 一句话：Hbase是山寨Google的Bigtable的开源、分布式、列数据库，可以解决海量数据的存储问题，适用于需要随机、实时读/写的场景；

# Hbase的出场背景

HBase 是Powerset在2007年创建的，最初是Hadoop的一部分。之后，它逐渐成为Apache软件基金会旗下的顶级项目，具备Apache软件许可证，版本为2.0。 

> 下面是一个HBase随时间发展的简短概述： 
>
> 2006年11月：Google发布BigTable论文。
>
> 2007年2月：HBase宣布在Hadoop项目中成立。
>
> 2007年10月：HBase第一个“可用”版本（Hadoop 0.15.0）。
>
> 2008年1月：Hadoop成为Apache的顶级项目，HBase成为Hadoop的子项目。
>
> 2008年10月：Hadoop 0.18.1发布。
>
> 2009年1月：HBase 0.19.0发布。
>
> 2009年9月：HBase0.20.0发布，性能有明显提升。
>
> 2010年5月：HBase成为Apache的顶级项目。
>
> 2010年6月：HBase 0.89.20100621，第一个开发版本。
>
> 2011年1月：HBase 0.90.0发布，稳定性和持续性有所提升。
>
> 2011年年中：HBase 0.92.0发布，支持协议处理器和安全控制。 
>
> 
>
> 2010年5月前后，HBase的开发者决定打破一直依赖的、步调一致的Hadoop的版本编号。原因是HBase有一个更快的发布周期，同时更接近1.0版本的水平，比Hadoop的预期更快。 
>
> 为此，版本号从0.20.x跳到了0.89.x，跳跃相当明显。此外，还做了一个决定，将0.89.x定为早期的开发版本。在0.89的基础上最终发布了0.90，即面向所有用户的稳定版。 

> Powerset 
>
> 公司位于旧金山，开发了一套用于互联网的自然语言搜索引擎。在2008年7越1日，微软公司收购了Powerset，之后Powerset放弃了对HBase开发的后续支持。

# Hbase解决的问题

传统关系数据库的设计不能满足海量数据的读写需求：扩容问题、写单点问题；

Hbase的特点：

* 可动态扩容，Hadoop/HDFS Integration
* 强一致性读写， Strongly consistent reads/writes
* 自动分片， Automatic sharding
* 失败自动切换，Automatic RegionServer failover

## Hbase与Hive的关系

> 联系

* Hbase和Hive都是Hadoop生态中的重要组件，数据都是存储在HDFS上的；

> 区别

* Hive时效性在秒-分钟级别，适合用来对一段时间内的数据进行分析查询，例如，用来计算趋势或者网站的日志；HiveSQL的执行可以基于Spark或者MapReduce；
* HBase时效性在毫秒（100ms）级别，适合`准实时查询`，例如 Facebook 用 HBase 进行消息和实时的分析。Hbase可以基于Phoenix SQL访问；

## Hbase与RDBMS的区别

|              | HBase                                                        | RDBMS                            |
| :----------- | :----------------------------------------------------------- | -------------------------------- |
| 硬件架构     | 类似于 Hadoop 的分布式集群，硬件成本低廉                     | 传统的多核系统，硬件成本昂贵     |
| 容错性       | 由软件架构实现，由于由多个节点组成，<br />所以不担心一点或几点宕机 | 一般需要额外硬件设备实现 HA 机制 |
| 数据库大小   | PB                                                           | GB、TB                           |
| 数据排布方式 | 稀疏的、分布的多维的 Map                                     | 以行和列组织                     |
| 数据类型     | Bytes                                                        | 丰富的数据类型                   |
| 事物支持     | ACID 只支持单个 Row 级别                                     | 全面的 ACID 支持，对 Row 和表    |
| 查询语言     | 只支持 Java API <br />（除非与其他框架一起使用，如 Phoenix、Hive） | SQL                              |
| 索引         | 只支持 Row-key，<br />除非与其他技术一起应用，如 Phoenix、Hive | 支持                             |
| 吞吐量       | 百万查询/每秒                                                | 数千查询/每秒                    |

# Hbase的应用场景

1.  `写密集型`应用：每天写多读少的应用，比如IM的历史消息，游戏的日志；
2. 查询场景明确且简单，如扫描一条记录或根据rowkey进行前缀扫描，因此Hbase需要根据查询需求设计RowKey；
3. 对性能和`可靠性`要求非常高的应用，由于HBase本身没有单点故障，`可用性`非常高。
4. 数据量较大，而且`增长量`无法预估的应用，HBase支持在线扩展，即使在一段时间内数据量呈井喷式增长，也可以通过HBase`横向扩展`来满足功能。

# Hbase的限制场景

* 不适合以`复杂查询条件`来查询数据的应用，原生HBase只支持基于rowkey的查询，对于HBase来说，单条记录或者小范围的查询是可以接受的，大范围的查询由于`分布式`的原因，在性能上没有保障；

* 不适合`事务要求高`的应用，如多表Join查询