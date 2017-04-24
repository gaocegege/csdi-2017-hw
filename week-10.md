# Week 10 - RDMA

```
116037910024  高策
```

## Paper

Fast In-memory Transaction Processing using RDMA and HTM, Xingda Wei, Jiaxin Shi, Yanzhe Chen, Rong Chen, Haibo Chen SOSP 2015

## Before Class Questions

>How does DrTM detect conflicts between remote reads and local writes?

在检测事务的冲突上，DrTM 使用了 HTM 和 RDMA 两种技术，HTM 是一个硬件的特性，在硬件级别提供了有限的事务性内存的支持。RDMA 是 Remote Direct Memory Access，提供了远程直接访问内存，不阻塞 CPU 的操作。

在处理事务冲突时，是在 transaction layer 去做的。对于远程的读和本地的写操作引起的冲突，最简单的方法是用 RMDA 锁住一个 remote record，不管是读还是写。但是这样会大大降低并行性，因此文章进行了一些改进，引入了基于租约的锁，来保证读共享。而在读和本地写产生冲突时，读会通过 RDMA abort 本地的 HTM 事务，从而避免冲突。

> Why DrTM has the deadlock issue and how does it avoid the deadlock?

首先，在 DrTM 的 fallback handler 中，不能像传统的实现那样，一个简单的锁就可以解决问题，而是 fallback handler 通过 2PL，对于任何 record 都是以 remote 的形式进行访问。这里就有可能产生死锁。因为涉及到 remote locks 的顺序。

为了避免这个问题，DrTM 声明了一个全局的释放和申请锁的顺序，避免了死锁的问题。

> What’s the limitation of DrTM?

首先，DrTM 没有做到很好的可用性，这是它最大的局限性。还有就是需要硬件特性的支持，导致在很多现有的硬件上没有办法完全复刻 DrTM 的工作，而需要一些适配性的工作。
