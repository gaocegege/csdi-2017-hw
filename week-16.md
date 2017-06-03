# Week 16 - Bugs

```
116037910024  高策
```

## Paper

Towards Optimization-Safe Systems: Analyzing the Impact of Undefined Behavior, Xi Wang, Nickolai Zeldovich, M. Frans Kaashoek, and Armando Solar-Lezama SOSP 2013

## Before Class Questions

> How does STACK identify unstable code?

STACK 有一个假设，是关于 Reach(e) 和 Undef(e) 的。

```
Δ = ∀e:Reach(e) → ¬Undef(e)
Reach(e): when to reach and execute code fragment e
Undef(e): when to trigger undefined behavior at e
```

STACK会在 Assumption Δ 被允许和不允许的情况下分别模拟编译。

1. 先模拟假设不成立的情况进行一次编译；
1. 模拟假设成立的情况进行一次编译；
1. 查看前两步的执行结果有没有区别，有区别的地方就是unstable code。

> What is the trade-off made in STACK for better scalability?

STACK 为了更好的 scalability，它会用 LLVM 来内联函数，并且在内联后的 individual 函数上进行检查。

在设计上，STACK为了使可扩展性更高，在计算 `Δ = ∀e:Reach(e) → ¬Undef(e)` 的时候做了一些近似运算，使最后得到的结果可能会漏掉一些unstable code。

> How do you evaluate the limitations of STACK?

STACK 的 limitations 有：

```
如果执行第二步时得不到准确的结果，那么会漏报一些unstable code；
如果执行第一步时得不到准确的结果，就会产生误报(false warning / false positive)。
目前 STACK 给出的 undefined behavior pattern 可能不齐全。
```

所以如果要 evaluate STACK 的 limitations，需要去寻找一个真实的小型系统，然后去运行 STACK，统计 STACK 找到的所有 unstable code，和人工分析下来得到的所有 unstable code 的情况作比对。来看有哪些漏报和误报，占比有多少。
