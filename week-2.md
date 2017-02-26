# Week 2 - Flash File System

```
116037910024  高策
```

## Paper

F2FS: A New File System for Flash Storage, Changman Lee, Dongho Sim, Joo-Young Hwang,
and Sangyeun Cho, Samsung Electronics Co., Ltd.. FAST 2015

## Before Class Questions

> Why does F2FS try to reduce the effects of random write? What techniques does it use?

A: 首先，随机写是非常常见的，论文中提到 Kim et al. 对在 NAND Flash 上的随机写情况进行了
量化分析，他发现 Facebook 的移动应用会产生比顺序写多 150% 的随机写操作。其次，Min et al. 
观察到对 SSD 的频繁随机写会产生一些内部碎片，降低 SSD 的读写性能。最后，随机写的性能是非常
差的，SSD 本身就不是对随机写非常友好。

F2FS 没有采用传统的 LFS 用来提高随机写效率的方法，而是采取了一种比较独特的日志策略，结合了
normal logging 和 threaded logging。在日志空间足够的情况下，会把随机写 delay 成顺序
写，在不足的情况下会进行 threaded logging，它对 random write 是不那么有友好的。不过只
发生在到达一个阈值，normal logging 不能正常工作的情况下。

> What is 'wandering tree' problem? How does F2FS mitigate it?

A: Wandering Tree 是一个在以 LFS 为代表的使用 Copy on Write 技术的文件系统上都存在的
一个问题，当文件的数据块被更新的时候是写到 log 的末尾，该数据块的直接指针也因为数据位置的改
变而更改，然后间接指针块也因为直接指针块的更新而更新。

F2FS 引入了一个新的表，Node Address Table(NAT)。首先需要了解 F2FS 的文件结构，在传统的
文件系统中，indirect node block 的表项是指向 direct node block 的，但是在 F2FS 并不
是这样的，indirect node block 是指向 NAT 的，这样每当一个 data block 被污染的时候，会
更新 direct block 和 NAT 中的表项，这样就防止了在 indirect node block 中的传播。

> What is the design rationale of multi-head logging? How does F2FS separate cold/hot data?

A: Multi-head logging 是充分利用 NAND Flash 的特性的产物。因为 NAND Flash 有着很好的
并行性质，在并行的时候没有频繁的管理指令操作，这使得 multiple logging 不会造成明显的性能损
失。所以 F2FS 可以根据数据的更新频繁程度做出区分进而进行 multi-head logging。F2FS 会根据
日志的并行数目来进行冷热数据的分离。总体而言数据分为 hot，warm 和 code 三种，而 block 分为
node 和 data 两种，因此一共有六种不同的 block：

| Type        | Temp.           | Objects  |
| ------------- |:-------------:|:-----|
| Node | Hot | Direct node blocks for directories |
| Node | Warm | Direct node blocks for regular files |
| Node | Cold | Indirect node blocks |
| Data | Hot | Directory entry blocks |
| Data | Warm | Data blocks made by users |
| Data | Cold | Data blocks moved by cleaning;<br>Cold data blocks specified by users;<br>Multimedia file data |

在 6 个日志全开的情况下，是如表中分类，在 4 个时会合并 code 和 warm 的数据，在 2 个时会合并所有只分 node 和 data。

## Reference

* [http://www.cnblogs.com/honpey/p/4808798.html](http://www.cnblogs.com/honpey/p/4808798.html)
* [https://www.kernel.org/doc/Documentation/filesystems/f2fs.txt](https://www.kernel.org/doc/Documentation/filesystems/f2fs.txt)
