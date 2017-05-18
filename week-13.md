# Week 13 - Network Function Virtualization

```
116037910024  高策
```

## Paper

The Click modular router, Robert Morris, Eddie Kohler, John Jannotti, M. Frans Kaashoek SOSP 1999

## Before Class Questions

> Why must Click provide both push' andpull' methods? Can we eliminate one of the two operations? How/Why?

Push 和 Pull 是两种连接方式。 Push 是从源 Element 到下游的元素。在 Push 连接方式里，上游的元素会提交一个包到下游的元素。Pull 是从目的地元素到上游元素。在 Pull 的连接方式里，是下游的元素发送包请求到上游的元素。Push 和 Pull的共同使用可以使包转发连接的适当终止，可以很好地解决路由器控制流问题。例如包调度的决定——选择哪个队列去请求一个包对于组合的 Pull 元素来说是非常容易实现的。另外，系统不应该向繁忙的转发接口发送包，否则，这个接口就必须存储包，并且路由器会失去处理这些包的能力（丢弃，修改优先级等）。这个约束可以由简单的给转发接口一个 Pull 输入实现。然后这个接口就可以控制包转发，并且可以在它准备好的时候请求包。

> One limitation of Click is the difficulty of scheduling CPU time among pull and push paths. Why is it difficult? What would you do to improve it?

Click 的调度器就是一个 Pull Element，它有多个输出，而只有一个输入。至于 CPU 时间调度的问题， 是说 Click 没办法处理多个设备同时接收或者发送数据的情况。目前的处理方法是 linux 处理大部分这样的调度，剩下的交由 Click 来做。最终所有这些都应该由一个单一的机制来控制。关于 improve 可以把 linux 里相关的逻辑作为一个 Element 引入，不知是否可行。

> How can batch processing be applied to Click? Be specific and consider the impact on latency and throughput.

APSys 2012 上有一篇论文『The Power of Batching in the Click Modular Router』，它尝试了 psio，netmap，PF_ring 等等开源的 IO batching 的工具，最后选择了 psio。同时为了实现计算的 batching，它修改了现有的 Click。

这样做提高了 throughput，但是加大了 latency。这也跟 batching 的程度有关，理论上来说是可以控制的。