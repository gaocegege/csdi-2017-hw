# Week 11

```
116037910024  高策
```

## Paper

Using One-Sided RDMA Reads to Build a Fast, CPU-Efficient Key-Value Store, Christopher Mitchell, Yifeng Geng, Jinyang Li USENIX ATC 2013


## Before Class Questions

> What makes in-memory key-value store systems suitable for RDMA optimization?

因为 In Memory 的 Key-value Store 的大多数请求都是读操作，因此对于 RDMA 来说，这样的特点使得其实现比较简单，只需要对 get 请求做修改就可以了，这样既可以利用 RDMA 的优点，又不需要对系统做过多的修改。

> How does Pilaf ensure the data consistency of 'get' operations? Does it have any problems?

利用了一个『Self verifying』的数据结构，它包括一个 root 和很多 pointer，然后会记录一个 checksum。client 通过检查 checksum 可以检测到读写不一致。当遇到了数据竞争时，client 会自动地进行重试操作。

文中提到有两个应用场景会有问题，一个是在 server 修改 hash table 的时候 client 也在读 hash table，这会导致 client 从不合法的内存地址读取内容。

另外是 client 的指针引用可能非法的。比如当 server 在删除一个 key-value 对的时候，client 自身维护的引用就会已经是失效的。

> Why does Pilaf need two round-trips to perform a 'get' operation? Can we reduce the number of round-trips to one using existing RDMA operations? Why/How?

因为涉及到两次读操作，一次是读哈希表，一次是读真正的 key-value 的内容。可以考虑合并两个内存块，但是这样应该会使得可以存储的空间变小。