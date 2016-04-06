


# Active Anti-Entropy


在一个类似 Riak 的、支持[集群](http://docs.basho.com/riak/2.1.3/theory/concepts/Clusters/)和[最终一致性](http://docs.basho.com/riak/2.1.3/theory/concepts/Eventual-Consistency/)的系统中，保存在不同节点上的对象副本，在发生节点异常，客户端并发更新，或者物理数据丢失和损毁，以及任何分布式系统注定需要面对事件的情况下，出现冲突问题是经常发生的情况。冲突会在下面两种情况下出现：

- 数据丢失，即一个节点持有对象的副本，而另一个节点上却没有，或者
- 数据存在差异，即在不同的节点上对象的值不同

Riak 提供了两种解决对象冲突的方法：read repair 和 active anti-entropy (AAE) 。
这两种冲突解决机制既适用于 Riak 中的普通 key/value 数据，也适用于[基于索引的搜索](http://docs.basho.com/riak/2.1.3/dev/advanced/search/#indexes)功能。


> 关于 AAE 和强一致性的说明

> 如果你希望针对全部或部分数据使用 Riak 的[强一致性](http://docs.basho.com/riak/2.1.3/theory/concepts/strong-consistency/)特性，你将需要激活 AAE 功能。    
> 具体操作指令可以在 [using strong consistency](http://docs.basho.com/riak/2.1.3/dev/advanced/strong-consistency/) 文档中找到。


### Read Repair vs. Active Anti-Entropy

在 Riak 1.3 版本之前，副本冲突只能通过 [read repair](http://docs.basho.com/riak/2.1.3/theory/concepts/glossary/#read-repair) 方式解决；
read repair 属于一种 passive anti-entropy 机制，只能在客户端读请求到达 Riak 时才起作用；
在 read repair 方式下，如果负责协调读请求的 vnode 发现目标对象在不同节点上的值不同，则 repair 过程将被触发执行。

使用 read repair 方式的**优点**之一是，其不需要任何类型的后台进程辅助工作，故能够省掉 CPU 资源占用；
而**缺点**就是，修复过程只针对客户端读请求命中的对象；而其它存在副本冲突的对象，在没有被客户端读请求命中的情况下不会被处理；

active anti-entropy (AAE) 子系统是在 Riak 的 1.3 版本中添加进来的，引入之后就可以通过持续运行在后台的进程进行冲突解决；
而之前的 read repair 方式，是不连续运行的；
AAE 子系统对于持有所谓“冷数据”的集群中非常有用，这种数据可能在很长时间内（数月或数年）不会被读取，因此，基于 read repair 方式几乎不能完成冲突解决；

尽管 AAE 默认被使能了，但可以在必要时进行关闭；
请参考文档 [managing active anti-entropy](http://docs.basho.com/riak/2.1.3/ops/advanced/aae/) 了解 AAE 的使用方式，以及如何配置和监控 AAE 。

### Active Anti-Entropy and Hash Tree Exchange

为了在无需占用过多资源的情况下，对不同副本中的对象值进行比较，Riak 采用了 [Merkle tree](https://en.wikipedia.org/wiki/Merkle_tree) 完成不同节点间的 hash 值交换；

采用这种类型的交换方式，使得 Riak 能够针对由 Riak 对象 hash 值构成的平衡树进行比较；
若差异发生在树层级的某个 high level ，则说明该 high level 下至少有一个 low level 中的值发生了变更；
AAE 会以递归方式，逐层比较整棵树，直到其准确定位到特定节点间哪些值存在差异；
这种处理方式的结果就是，AAE 能够高效的进行修复操作，而不管集群中究竟保存了多少对象，因为其只需要修复特定的对象，而不是全部对象；

和其他类似系统相比，Riak 使用了持久的，基于磁盘的 hash tree ，而没有采用纯内存 hash tree 。好处有以下两点：

- Riak 在运行 AAE 操作的时候对内存使用的影响最小；
- Riak 节点的重启无需重建 hash tree

除此之外，hash tree 是在新的写请求到来时实时更新的，这将会减少 Riak 检测并修复副本差异的整体时间；

作为一种额外的应变措施（fallback）措施，Riak 会周期性的清理 hash tree ，并根据磁盘上的 key/value 重建所有 hash tree ；
这种模式有助于 Riak 检测到由于磁盘失效、硬件错误，和其他原因导致的 slient 磁盘数据损毁；
该重建行为的默认时间周期为一周，但是可以在每一个节点上的[配置文件](http://docs.basho.com/riak/2.1.3/ops/advanced/configs/configuration-files/#active-anti-entropy)中进行单独调整。