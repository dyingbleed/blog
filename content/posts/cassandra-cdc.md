---
title: "Cassandra CDC"
date: 2023-07-15T07:42:48+08:00
draft: false
categories: ["编程"]
tags: ["cassandra"]
---

# 开启 Cassandra CDC

开启 Cassandra CDC 需要同时开启表级别 CDC 和节点级别 CDC

开启表级别 CDC，更新表属性 `cdc`：

```
ALTER TABLE foo WITH cdc=true;
```

开启节点级别 CDC，更新配置文件 `cassandra.yaml`：

```
cdc_enabled=true
```

其他 CDC 相关配置：

- `cdc_raw_directory`
- `cdc_total_space`
- `cdc_free_space_check_interval`

**WARNNING** 开启 Cassandra CDC 之后，当节点的 `cdc_total_space` 已满，Cassandra 会拒绝写入开启了 CDC 的表

# Cassandra 3.11.X CDC 实现

为了理解 Cassandra CDC, 需要对 [LSM-Tree (Wikipedia)](https://en.wikipedia.org/wiki/Log-structured_merge-tree) 有所了解

![Cassandra LSM-Tree](https://dyingbleed-cn.oss-rg-china-mainland.aliyuncs.com/blog/cassandra_lsm-tree.jpg)

Cassandra 的 memtable 是按表分配的，Cassandra 的 commitlog 是全局的并由多个 commitlog segment 文件组成

每个 memtable 维护了 low commitlog position 指向该 memtable 所有记录对应 commitlog 的最小位置

当满足 $last\\\_position_{segment} < \min(low\\\_segment\\\_position_{memtable})$ 条件，则 commitlog segment 可以被安全的作废

在 Cassandra 3.11.X 版本，作废的 commitlog segment 文件如果包含了 CDC 数据，就会从 `commitlog_directory` 目录移动到 `cdc_raw_directory` 目录，否则会被删除

![Commitlog](https://dyingbleed-cn.oss-rg-china-mainland.aliyuncs.com/blog/cassandra_3_cdc.jpg)

# Cassandra 4.X CDC 实现

TBD

# Debezium Cassandra Connector

TBD

# References

- [Change Data Capture | Apache Cassandra Documentation](https://cassandra.apache.org/doc/latest/cassandra/operating/cdc.html)
- [Understanding an Apache Cassandra Memtable Flush | A Bias For Action](https://abiasforaction.net/apache-cassandra-memtable-flush/)
- [Learn How CommitLog Works in Apache Cassandra | Apache Cassandra Documentation](https://cassandra.apache.org/_/blog/Learn-How-CommitLog-Works-in-Apache-Cassandra.html)