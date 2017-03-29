# Week 6 - Transactional Memory/DB

```
116037910024  高策
```

## Paper

Transactional memory: architectural support for lock-free data structures, Maurice Herlihy, J. Eliot B. Moss ISCA 1993

## Before Class Questions

>What is lock-free data structure? Why do we need lock-free data structure?

Lock-free 是一个在并发场景下的概念。在传统的并发时，为了实现多个核之间共享数据，往往需要在共享数据时使用互斥的操作来保证在同一时间内只有一个核可以对数据进行写入。而所谓的 lock-free 数据结构就是指在处理并发的时候没有互斥的操作。

Lock-free 的数据结构具有一些好处，文中提到可以解决优先级、Convoying 和死锁的问题，同时可以提高性能。

>What is orphan transaction? Why do we need VALIDATE instruction?

Orphan transaction 是指已经被 abort 的进程仍然在执行的事务，通常是因为事务因为一些错误，导致一直在尝试获取资源，但是没办法成功导致事务在无限 retry 当中的现象。

>Open question: What is the shortcoming of transactional memory compared with conventional locking techniques? Do you have any suggestion to mitigate those problems?

**Disadvantages**

事务性内存对于 short critical section 的场景下使用比较合适，在大的时候，会使得事务比较频繁地被中止。这也是论文中提到的。除此之外，事务性内存也使得开发者的思维要一定程度的转变，不是很容易被接受。而且目前大多数代码都是使用锁的方式来实现的，代码迁移成本很高。

**Suggestions**

对于前者，可以向文中所说的那样加入对应的硬件支持。对于后者，可以实现一个 lock to tm 的中间层，这样对于 tm 的接受度会高很多。