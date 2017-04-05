# Week 9 - Distributed Transactions

```
116037910024  高策
```

## Paper

Spanner: Google’s Globally-Distributed Database, James C. Corbett, Jeffrey Dean, etc. OSDI 2012

## Before Class Questions

> What is external consistency? What’s the difference between external consistency and serializability?

External consistency 在文中的描述是：如果事务 2 发生在事务 1 提交之后，那事务 2 的时间戳要比事务 1 提交的时间戳要大。

Serializability 是数据库隔离性上的一个级别，这意味着数据库中所有的事务都是可以被序列化来执行的，只有完全没有冲突的事务才可以并发地执行。

External consistency 是一致性上的概念，在分布式场景下，external consistency 是更难做到的，因为时序对时间的精度要求很高，在分布式场景下，有可能出现因为不同机器系统时间不一致导致事务 2 拿到一个比事务 1 提交的时间戳更小的时间戳。

Serializability 是隔离性上的概念，如果做到了 external consistency，就一定可以做到 serializability。

> How does Spanner achieve the external consistency?

 Spanner 之前的 Percolator 和 Spanner 都是使用全局的时钟来解决 external consistency 的。但是 Spanner 创新地使用了原子钟和 GPS 来作为全局的时钟，以此来实现 external consistency。

 在事务的执行中，Spanner 会保证，每个事务的 commit timestamp 都会在其 start 和 commit 之间。Spanner 依赖的底层容器集群系统 Borg 会维护一个 True Time API，这个 API 会返回精度为 ε 的时间区间 `[t - ε, t + ε]`。因此每个事务会在 start 和 commit 的时候分别调用一次 True Time API，拿到两个时间区间 `[t1 - ε,t1 + ε]` 和 `[t2 - ε,t2 + ε]`，因此在区间 `[t1 + ε,t2 - ε]` 之间的时间都是可用的，如果 t1 和 t2 很接近，那最多需要等 2ε。

> What will happen if the TrueTime assumption is violated? How the authors argue that TrueTime assumption should be correct?

这会导致 True Time API 不能再用来保证 external consistency，文章中提到，CPU 造成的问题比时钟问题多六倍，因此跟硬件造成的错误相比，时钟造成的问题不值一提，可以被视为是值得信任的。
