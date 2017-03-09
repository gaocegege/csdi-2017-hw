# Week 4 - Crash Recovery

```
116037910024  高策
```

## Paper

Fast Crash Recovery at RAMCloud, Diego Ongaro, etc SOSP 2011

## Before Class Questions

>Why RAMCloud uses a log-structured strategy for data in DRAM?

RAMCloud 在备份上，没有完全使用内存做备份，而是在内存中有一个备份，在硬盘上也有一个备份。这样的备份策略省了内存，但是造成了两个问题：

* 备份相对慢速，可能会影响系统的正常操作
* 系统 crash 后恢复要从硬盘中开始，会比较慢

Log-Structured Strategy 就是为了解决第一个问题而采用的。请求到了内存里是以日志的形式记录，随后会将 entry 分发给各个备份，各个备份在收到后会直接返回，然后再处理，约等于实现了异步的操作。因此备份时的 overhead 被 hide 了，但是这样引入了新的问题，可能导致系统 crash 后没有把 buffer flush 到真正的存储中。软件不行硬件来凑，文中提到了两种方法来保证 entry 到了 buffer 中就是持久化的，一种是 DIMM + super-capacitor，一种是加电池。

综上，日志主要是为了解决同步备份的时候因为 hierarchy 产生的性能问题。

>Which policy does RAMCloud use to place segment replicas and how to find the segment replicas during recovery?

以往的实现是中心化的 coordinator，这样会造成性能瓶颈。所以 RAMCloud 采纳了去中心化的思想，利用了随机化和微调的方式，来分散备份。有些类似 k choises 的选择，RAMCloud 会先随机选择一些，然后再从中选出最合适的，并且加入了 reject 的机制来保证在乐观并发的情况下不会产生竞争的问题。同时为了保证尽可能贴近最优解，RAMCloud 考虑了硬盘速度以及硬盘上已经有的 segment 的数量来进行微调，保证尽可能的均匀。

recovery 的时候以往的实现是在 coordinator 里维护一个中心化的表，这样的做法向上面提到的一样会造成瓶颈，因此 RAMCloud 在 recovery 的时候会问所有的备份，备份会返回一个它存储的副本列表，整个过程是并行的而且 RAMCloud 用了自称 fast 的 RPC，因此整个过程不会特别慢。

>Does RAMCloud support random access? If so, please explain how.

支持不支持，支持。

RAMCloud 在每个 master 上都维护有一个哈希表，结构为 `<table identifier, object identifier>`，通过哈希表，可以进行随机的访问。