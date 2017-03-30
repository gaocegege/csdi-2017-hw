# Week 8 - NoSQL

```
116037910024  高策
```

## Paper

[Bigtable: A Distributed Storage System for Structured Data](http://static.googleusercontent.com/media/research.google.com/en//archive/bigtable-osdi06.pdf)

## Before Class Questions

> What is the relationship between table , tablet and sstable?

Table 就是用户可见的表，它有 Row，Column，和不同 Version 的 Value。Table 会从逻辑上划分为多个 Tablet 以方便存储。

Tablet 是一个存储单元。在 BigTable 的架构中，是由一个 Master Server 和多个 Tablet Server 组成的。而 Tablet 这一层抽象有些类似传统数据库概念中的 sharding，跟在 HBase 中的 Region 是差不多的思想。Tablet 的读写是由 Tablet Server 来处理的。

SSTable 是一种文件的格式，全称是 "Sorted Strings Table"，是指按照 Key 的排序在文件中存储 <Key, Value> 对。在逻辑上 SSTable 会包含多个 Block，每个 Block 为了方便寻址会有一个 index，所有的 Block index 会写在 SSTable 文件的最后，每当 SSTable 文件被 open 的时候，会将索引加载到内存里，这样每次 Lookup 的时候只有一次硬盘读取。SSTable 也支持全部读取到内存里，这样在 Lookup 的时候没有任何硬盘的读取。这个跟 HBase 中 HFile 的文件格式实现很相似。

> Describe what will happen when a read operation or write operation arrives.

读取和写入是以 Tablet 作为一个 Unit 进行的。

在进行写操作的时候，Tablet Server 会先做一些检查，保证请求的合法以及权限问题。权限的检查是通过检查一个在 Chubby 中的列表进行的，这个列表会被 Client Library 缓存住。一次被允许的写操作会先进入 Commit Log，在处理 Log 的时候采取了批处理来提高吞吐。在操作被 Commit 后，它的内容会被插入 MemTable 里，当 MemTable 的 size 超过一个阈值的时候，会让当前的 MemTable 进入一个 frozen 的状态，随后创建一个新的 MemTable，Frozen 的 MemTable 就可以以 SSTable 的形式写入 GFS。

在进行读操作的时候，Tablet Server 会在做了一些检查保证合法后，在 MemTable 和 SSTable 的一个 merge 后的 view 中来进行读操作，这样可以保证可以读到最新的值。

> Describe which applications are BigTable suitable for and not suitable for.

BitTable 适合那些对可用性要求比较高的业务场景，同时对于跨行的事务性没有要求的应用。但是正因为没有跨行事务的支持，所以我觉得引用场景很局限。目前在谷歌内部应该也逐渐被 Spanner 和 F1 所取代吧。
