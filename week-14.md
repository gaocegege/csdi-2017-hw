# Week 14 - Graph Query on RDF

```
116037910024  高策
```

## Paper

Fast and Concurrent RDF Queries with RDMA-based Distributed Graph Exploration, Jiaxin Shi, Youyang Yao, Rong Chen, Haibo Chen, Feifei Li OSDI 2016

## Before Class Questions

> What are the bottlenecks of existing RDF systems?

一共有两种不同的实现，分别是 Triple store and triple join 和 Graph store and graph exploration。前者是以 triple 的方式来将 RDF 数据存储在关系型数据库中，因此查询有两个步骤，scan 和 join。scan 会分为子查询，最后再借由 hash join 之类的 join 的操作将查询的结果 join 在一起。由此可知如果数据非常大的时候，最后的 join 会是很大的问题。

第二种方式是以图的方式来存储和查询 RDF。这样的方式以 Trinity.RDF 为代表，有一些剪枝的优化。但是最后也会有一个 final join 的过程。

纵观之前的实现，最后的 join 是一个最大的问题。

> What are the differences between Wukong and prior graph-based designs? What are the benefits?

最大的不同在于索引的存储方式。之前的基于图的设计都是用独立的索引数据结构来存索引，但是 Wukong 是把索引同样当做基本的数据结构（点和边）来存储。并且会考虑分区来存储这些索引。

这样做有两个好处，第一点就是在进行图上的遍历或者搜索的时候可以直接从索引的节点开始，不用做额外的操作。第二点是这样使得索引的分布式存储变得非常简单，复用了正常的数据的存储方式。

> What is full-history pruning and what's the difference compared with the prior pruning approach? Why can Wukong adopt full-history pruning?

Full-history 就是说所有的历史记录都会被记录下来。之前是只记录一次的。之所以可以这样做是因为一方面 RDF 的查询都不会有太多步，而且 RDMA 在低于 2K bytes 的时候性能都是差不多的，所以 Wukong 可以这样做。
