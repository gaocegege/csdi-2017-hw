# Week 3 - Non-volatile Memory

```
116037910024  高策
```

## Paper

System software for persistent memory, Subramanya R Dulloor, Sanjay Kumar, Anil Keshavamurthy, Philip Lantz, Dheeraj Reddy, Rajesh Sankaran and Jeff Jackson EuroSys 2014

## Before Class Questions

>Why do memory reordering and cache play an important role in PMFS consistency? How does PMFS maintain consistency?

因为文件系统的 recovery 需要依赖写入的 order。因为 PMFS 在元数据的记录中使用了 journal 的方式，因此在写入 log 的时候需要保证数据写入的顺序才能保证日志能够完整地记录所有的 commit 以保证 consistency。而 reorder 会导致不可预测的日志状态，所以 order 是非常重要的。一般来说是用 cflush 来做的，但是 cflush 是从 cache 到 memory 并不能保证从 memory 中的 controller 到真实的内存中，cache 的作用也在此。

PMFS 引入了一个硬件原语：pm_wbarrier，用所谓细粒度的 log 来实现  consistency。在可以使用原子写的地方，PMFS 会率先使用原子写。对于 meta data 而言 PMFS 使用了 journal 的方式，因为 meta data 的 size 比较小。对于真正的 data 会使用 copy on write 的方式来保证一致性。在 PMFS 没有被 umount 的时候断电， PMFS 会在下一次 mount 的时候进行 recovery。对于 PMFS 的 allocator 的 consistency，会遍历文件系统的 B 树来进行 recovery。这几方面综合就是 PMFS 为了解决 consistency 问题做的事情。

>There are three techniques to support consistency: copy-on-write, journaling and log-structured updates. Please explain the differences among them. Which technique does PMFS choose?

Copy on write 类似于 F2FS 中的对 NAND Flash 的做法，在写的时候会复制一个数据副本，然后在副本上进行修改，最后再将指针指向副本所在的地址。Journaling 的典型实现是 EXT 4，在存储数据之前，会先将数据元数据等存放在 log 中，在事务提交后才将数据真正写入。在 recovery 的时候会把 log 里的所有事务重放一遍。Log-structured updates 是 F2FS 基于的 recovery 方式，相比于 journaling 的多次写，LFS 只需要一次追加写。

如上题所述，PMFS 采取了一种 hybrid 的实现方式，由本地原子写，CoW 和 journaling 共同实现了 consistency。

*『在可以使用原子写的地方，PMFS 会率先使用原子写。对于 meta data 而言 PMFS 使用了 journal 的方式，因为 meta data 的 size 比较小。对于真正的 data 会使用 copy on write 的方式来保证一致性。在 PMFS 没有被 umount 的时候断电， PMFS 会在下一次 mount 的时候进行 recovery。对于 PMFS 的 allocator 的 consistency，会遍历文件系统的 B 树来进行 recovery。这几方面综合就是 PMFS 为了解决 consistency 问题做的事情。』*

>What granularity does PMFS use for log? Please explain the reason.

PMFS 针对上述的三种技术，进行了成本的分析，主要是分析写入的数据大小和 pm_wbarrier 操作的次数，发现在对 metadata 的更新的时候，将每个 log entry 设为 64 byte，跟 cacheline 一样，有最小的 overhead。
