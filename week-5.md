# Week 5 - Multiprocessor, Concurrency & Locking

```
116037910024  高策
```

## Paper

Non-scalable locks are dangerous, Silas Boyd-Wickizer, M. Frans Kaashoek, Robert Morris, and Nickolai Zeldovich Proceedings of the Linux Symposium. 2012.

## Before Class Questions

>Why does the performance of ticket lock collapse with a small number of cores? (Hint: cache coherence protocol)

在 ticket lock 的实现中，是维护两个 ticket 变量，当前 ticket 和 next ticket。在 unlock 操作中是将当前 ticket 自增从而使得下一个拿到了 ticket 的 core 获取到了锁。

凡是遵循文章中提出的硬件缓存一致性模型的系统，都会遇到 ticket lock 的 rapid collapse。文章提出的硬件缓存一致性模型用目录的方式类比了 CPU 中的缓存，并且用马尔可夫链模型，对 ticket lock 的问题进行了分析。其中每一个节点代表有几个 core 在等的状态，Arrival and Service Rates 分别代表了在不同状态下锁的获得和释放。其中到达率跟在没有在等的 core 的数量 (n - k) 成正比，服务率和在等的 core 的数量 (k) 成反比。所以随着 k 的增大，服务率是减小的。这使得模型得到了一个数学上的结论，锁的获取时间和正在等待锁的 core 的数量是成正比的。

因此随着 core 的增加，在 serial section 很小的时候，Sk = 1/(s + ck/2) 中 k 对 Sk 的贡献越大，因此越容易受到 k 的影响。这也就是为什么，在 serial section 很小的时候，只是多了很少一些在等待锁的 cores，就出现了性能的雪崩。

>To mitigate the performance collapse problem of ticket lock, we can replace it with MCS lock. Please descibe the MSC lock algorithm in C.

```c
struct QNode {
	QNode* next;
	bool locked;
}

// parameter I, below, points to a qnode record allocated
// (in an enclosing scope) in shared memory locally-accessible
// to the invoking processor

void acquire_lock(QNode** L, QNode* I) {
	QNode* predecessor;
	I->next = NULL;
	// fetch_and_store atomically writes I to L and returns previous value of L
	predecessor = fetch_and_store(L, I);
	if (predecessor != NULL) {
		I->locked = true;
		predecessor->next = I;
		// Spin
		while(I->locked);
	}
}

void release_lock(QNode** L, QNode* I) {
	if (I->next == NULL) {
		// compare_and_store returns true iff it stored
		if (compare_and_store(L, I, NULL)) {
			return;
		}
		while(I->next == NULL);
	}
	I->next->locked = false
}
```

>Why does MCS lock have better scalability than ticket lock?

在 ticket lock 中，所有的 core 都去依靠同一个变量来获得锁，而在 MCS lock 中，前一个 core 在 release 的时候会修改下一个 core 的 locked 变量，通知下一个 core 来获得锁。这样每个锁都只依靠属于自己的一个变量，这样的实现是缓存一致性无关的，有着更好的 scalability。
