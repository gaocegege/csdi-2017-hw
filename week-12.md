# Week 12 - Low (Tail) latency

```
116037910024  高策
```

## Paper

Dean, Jeffrey, and Luiz André Barroso. "The tail at scale." Communications of the ACM 56.2 (2013): 74-80.

## Before Class Questions

> Why latency variability is amplified by scale?

这是受到系统的特性影响的。在文中给出的例子，是假如一个用户访问一个服务器，99% 的请求是在 10ms 内被处理完的，而有 1% 的请求是在 1s 内被处理的。在这种情况下，针对一个服务器的请求还是可以接受的。但是现在绝大多数应用都是需要多个服务共同服务最后才返回给终端用户的，因此在这样的情况下，用户感知到的延迟是所需要的服务中延迟最大的值。因此，在这样的情况下，用户感知到的延迟是随着系统规模的增大而提高的。这里的 scale 指的是系统的规模。

Please briefly outline some effective tail-tolerant techniques.

比如最简单的，使用多台服务器，处理完全相同的逻辑，用 replica 的方式降低延迟。用户的请求会发送给所有的服务器，当任何一个服务器处理完请求，就返回。这样的方式要求应用要自己处理幂等请求，逻辑也会变得复杂一些。在之上可以做一些优化，比如在一个服务器处理完请求后，通知其他服务器终止对幂等请求的处理等等，不过这样也更加增加了实现的难度。

还有一些减少 daemon 进程的方法比如使用 unikernel 等等更加精简的内核，但是这样的做法成本就更高了。

Why tolerating latency variability for write operations is easier?

因为一般写都是异步的，而且可以容忍一定的不一致。以及在一致性的写上一般都会使用 Paxos, ZAB, raft 之类的算法。而这些算法本身就是 tail tolerant 的。
