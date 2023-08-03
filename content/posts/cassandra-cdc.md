---
title: "Cassandra CDC"
date: 2023-07-15T07:42:48+08:00
draft: false
categories: ["编程"]
tags: ["cassandra"]
---

# Enable Cassandra CDC

To enable Cassandra CDC, need to enable both the table-level CDC and node level CDC.

To enable table-level CDC, update the table option `cdc`：

```
ALTER TABLE foo WITH cdc=true;
```

To enable node-level CDC, update the configuration file `cassandra.yaml`：

```
cdc_enabled=true
```

Other CDC options：

- `cdc_raw_directory`
- `cdc_total_space`
- `cdc_free_space_check_interval`

**WARNNING** If CDC is enabled, when fill up the `cdc_free_space_in_mb`, writes to CDC-enabled tables will be rejected.

# Cassandra 3.11.X CDC Implementation

![Cassandra LSM-Tree](https://dyingbleed-cn.oss-rg-china-mainland.aliyuncs.com/blog/cassandra_lsm-tree.jpg)

Memtable is per table (or column family).

Commitlog is global, and consisted by commitlog segments.

Memtable maintain two positions:

- `commitlogUpperBound` low commitlog position;
- `commitlogLowerBound` high commitlog position;

Commitlog segment maintain two hash maps:

- `cfDirty` table (or column family) unflushed data interval;
- `cfClean` table (or column family) flushed data interval;

When keyspace apply new mutation:

1. append mutation to commitlog
2. commitlog segment update `cfDirty` table
3. add partition update to memtable
4. memtable update `comitlogUpperrBound`

When flush memtable:

1. iterate over active commitlog segments, mark clean using `commitlogUpperBound` and `commitlogLowerBound`

In Cassandra 3.11.X version, if commitlog segment contains CDC-enabled table mutation, the commitlog segment will be moved from `commitlog_directory` to `cdc_raw_directory`.

![Commitlog](https://dyingbleed-cn.oss-rg-china-mainland.aliyuncs.com/blog/cassandra_3_cdc.jpg)

# Cassandra 4.X CDC Improvement

TBD

# Debezium Cassandra Connector

TBD

# References

- [Change Data Capture | Apache Cassandra Documentation](https://cassandra.apache.org/doc/latest/cassandra/operating/cdc.html)
- [Understanding an Apache Cassandra Memtable Flush | A Bias For Action](https://abiasforaction.net/apache-cassandra-memtable-flush/)
- [Learn How CommitLog Works in Apache Cassandra | Apache Cassandra Documentation](https://cassandra.apache.org/_/blog/Learn-How-CommitLog-Works-in-Apache-Cassandra.html)