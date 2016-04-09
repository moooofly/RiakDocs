
# Riak 术语表


下面的内容是研读 Riak 文档时经常遇到的术语的列表；同时给出了指向更详细信息的链接地址；

#### Active Anti-Entropy (AAE)
A continuous background process that compares and repairs any divergent, missing, or corrupted replicas. Unlike [read repair](http://docs.basho.com/riak/2.1.3/theory/concepts/Replication/#Read-Repair), which is only triggered when data is read, the Active Anti-Entropy system ensures the integrity of all data stored in Riak. This is particularly useful in clusters containing “cold data,” i.e. data that may not be read for long periods of time, potentially years. Furthermore, unlike the repair command, Active Anti-Entropy is an automatic process requiring no user intervention. It is enabled by default in Riak 1.3 and greater.

[Replication](http://docs.basho.com/riak/2.1.3/theory/concepts/Replication/#Active-Anti-Entropy-AAE-)


#### Basho Bench
Basho Bench 是一种基准测试工具，用于生成精准的、可重复的性能测试和压力测试行为，以便绘制相应的性能图。

[Basho Bench](http://docs.basho.com/riak/2.1.3/ops/building/benchmarking/)
[GitHub repository](https://github.com/basho/basho_bench/)

#### Bucket
bucket 是指 Riak 中用于数据存储的命名空间，可以用于针对其保存的内容设置一系列通用属性；例如，副本数目（n_val），读操作发生时 sibling 上的内容是否被返回等等；
bucket 的属性是由 bucket type 决定的；

[Buckets](http://docs.basho.com/riak/2.1.3/theory/concepts/Buckets/)
[HTTP Bucket Operations](http://docs.basho.com/riak/2.1.3/dev/references/http/#Bucket-Operations)


#### Bucket Type
Bucket type 的作用在于，允许你创建和管理一系列 bucket 属性集，当作用于 bucket 上时，等于隐式确定了 bucket 的行为模式；
Bucket type 也可以作为 Riak 中除 bucket 和 key 之外的第三种命名空间使用；

[Bucket Types](http://docs.basho.com/riak/2.1.3/dev/advanced/bucket-types/)

#### Cluster
Riak 集群是由 160-bit 整数空间等分后的分区构成的；
位于 Riak Ring 中的每一个 vnode 负责一个上述分区；

[Clusters](http://docs.basho.com/riak/2.1.3/theory/concepts/Clusters/)
[Dynamo](http://docs.basho.com/riak/2.1.3/theory/dynamo/)


#### Consistent Hashing
一致性哈希是一种 当 hash table 中的数据结构需要重新进行平衡时（即发生 slot 添加或移除时），用于限制 reshuffling keys 数量的 技术；
Riak 使用一致性哈希组织数据的存储和复制；
特别要指出的是，负责对象存储的 Riak Ring 中的 vnode 是一定需要使用该技术的；

[Clusters](http://docs.basho.com/riak/2.1.3/theory/concepts/Clusters/)
[Dynamo](http://docs.basho.com/riak/2.1.3/theory/dynamo/)
[Wikipedia:Consistent Hashing](https://en.wikipedia.org/wiki/Consistent_hashing)

#### Data Types
Riak 中的数据类型是指这样一种数据对象：

- 该数据对象受启发于对 [CRDT](http://hal.upmc.fr/file/index/docid/555588/filename/techreport.pdf) 的研究；
- 采用了一定的收敛规则以决定副本间的冲突在 Riak 的最终一致性系统里如何得以解决；
- 目前存在 5 种 Riak 数据类型：flags, registers, counters, sets 和 maps ；

[Data Types Concept](http://docs.basho.com/riak/2.1.3/theory/concepts/crdts/)
[Using Data Types](http://docs.basho.com/riak/2.1.3/dev/using/data-types/)
[Data Modeling with Riak Data Types](http://docs.basho.com/riak/2.1.3/dev/data-modeling/data-types/)


#### Eventual Consistency
最终一致性模型是指，在没有发生针对指定数据条目的、新的更新时，对于该条目的所有读操作都将获取到最后更新的值内容；
在 Riak 中实现的最终一致性细节信息可以查阅下面的问题。

[Eventual Consistency](http://docs.basho.com/riak/2.1.3/theory/concepts/Eventual-Consistency/)


#### Gossiping
Riak 使用 gossip 协议在集群中分享和传播 ring 的状态，以及 bucket 属性信息；
无论何时，只要一个节点变更了其对 ring 中管辖区域的声明，就会通过该协议通知此变动；
每个节点还会周期性的、向随机选择的其他节点发送自身环状态的当前视图，以防有节点错过了之前的更新；

[Clusters](http://docs.basho.com/riak/2.1.3/theory/concepts/Clusters/)
[Adding and Removing Nodes](http://docs.basho.com/riak/2.1.3/ops/running/nodes/adding-removing/#The-Node-Join-Process)

#### Hinted Handoff
Hinted handoff 是指这样一种技术：当 Riak 集群中存在节点失效的情况时，会使用失效节点的邻居节点临时性的接管针对该失效节点的存储操作；
当失效节点恢复正常后，再由邻居节点将接收并处理的更新内容传递给它；

Hinted handoff 令 Riak 能够确保数据库的可用性；即使目标节点失效了，Riak 仍然能够继续处理请求；

[Recovering a Failed Node](http://docs.basho.com/riak/2.1.3/ops/running/recovery/failed-node/)


#### Key
key 是 Riak 中的唯一对象标识符；
key 的作用域在 bucket 和 bucket type 之下；

[Keys and Objects](http://docs.basho.com/riak/2.1.3/theory/concepts/keys-and-values/)
[Developer Basics](http://docs.basho.com/riak/2.1.3/dev/using/basics/)


#### Lager
[Lager](https://github.com/basho/lager) 是一种 Erlang/OTP 框架，用作 Riak 的默认日志处理器；

#### MapReduce
Riak 实现的 MapReduce 功能为开发者提供了针对 key/value 数据的、更加强悍的查询能力；

[Using MapReduce](http://docs.basho.com/riak/2.1.3/dev/using/mapreduce/)
[Advanced MapReduce](http://docs.basho.com/riak/2.1.3/dev/advanced/mapreduce/)

#### Node
一个节点类似于一台物理服务器；
每一个节点会映射到 Riak ring 的键空间中的一个分区上；

[Clusters](http://docs.basho.com/riak/2.1.3/theory/concepts/Clusters/)
[Adding and Removing Nodes](http://docs.basho.com/riak/2.1.3/ops/running/nodes/adding-removing/)


#### Object
对象实际上是 value 的别名；

[Keys and Objects](http://docs.basho.com/riak/2.1.3/theory/concepts/keys-and-values/)
[Developer Basics](http://docs.basho.com/riak/2.1.3/dev/using/basics/)

#### Partition
分区是指 Riak 集群按照某种规则划分后的空间；
Riak 中的每一个 vnode 负责一个分区；
数据会被保存到数量为 n_val 的一组分区中，而目标分区的选定是通过对每个对象的 key 值进行一致性哈希计算得到的；

[Clusters](http://docs.basho.com/riak/2.1.3/theory/concepts/Clusters/)
[Eventual Consistency](http://docs.basho.com/riak/2.1.3/theory/concepts/Eventual-Consistency/)
[Cluster Capacity Planning](http://docs.basho.com/riak/2.1.3/ops/building/planning/cluster/#Ring-Size-Number-of-Partitions)

#### Quorum
Quorum 在 Riak 中具有两层含义：

- 第一层含义：认定 read 或 write 请求操作成功前，必须成功进行回应的副本数量；该值或者被定义在 bucket 属性中，或者作为一条单独请求(R,W,DW,RW)的相关参数出现；
- 第二层含义：代表上述描述中的数值的符号，等于 n_val / 2 + 1 ，默认设置时为 2 ；

[Eventual Consistency](http://docs.basho.com/riak/2.1.3/theory/concepts/Eventual-Consistency/)
[Replication Properties](http://docs.basho.com/riak/2.1.3/dev/advanced/replication-properties/)
[Understanding Riak's Configurable Behaviors: Part 2](http://basho.com/riaks-config-behaviors-part-2/)

#### Sloppy Quorum
Sloppy Quorum 可以认为是不严格的 Quorum ；
在节点失效的场景中，可用节点数 < 总节点数，此时基于 Sloppy Quorum 机制，则可以确保 Riak 仍旧能够接受写请求；
当主（primary）节点不可用时，另外的节点将会接收相应的写请求；当之前的节点恢复后，数据会被通过 [Hinted Handoff](http://docs.basho.com/riak/2.1.3/theory/concepts/glossary/#Hinted-Handoff) 处理过程传回到主节点；

#### Read Repair
Read repair 属于一种 anti-entropy 机制，当出现针对陈旧数据的读请求时，Riak 会通过该机制更新陈旧副本；

[More about Read Repair](http://docs.basho.com/riak/2.1.3/theory/concepts/Replication/)

#### Replica
副本是指保存在 Riak 中的数据的版本；
在 Riak 中，读和写操作的成功取决于副本数量的配置；
该值的配置需要考量应用具体的一致性和可用性要求；

[Eventual Consistency](http://docs.basho.com/riak/2.1.3/theory/concepts/Eventual-Consistency/)
[Understanding Riak's Configurable Behaviors: Part 2](http://basho.com/posts/technical/riaks-config-behaviors-part-2/)

#### Riak Core
Riak Core 是模块化的分布式系统框架，用作 Riak 可扩展架构的基础；

[Riak Core](https://github.com/basho/riak_core)
[Where To Start With Riak Core](http://basho.com/where-to-start-with-riak-core/)


#### Riak KV
Riak KV 是 Riak 中针对 key/value 的数据存储器；

[Riak KV](https://github.com/basho/riak_kv)

#### Riak Pipe
Riak Pipe 是 Riak 用于支持 MapReduce 功能的处理层；最好将其理解成 UNIX pipes for Riak ；

[Riak Pipe](https://github.com/basho/riak_pipe)
[Riak Pipe - the New MapReduce Power](http://basho.com/posts/technical/riak-pipe-the-new-mapreduce-power/)
[Riak Pipe - Riak's Distributed Processing Framework](http://vimeo.com/53910999)


#### Riak Search
Riak Search 是一种分布式、可扩展、容错性、实时、全文搜索引擎；
Riak Search 将 [Apache Solr](https://lucene.apache.org/solr/) 集成到了 Riak KV 中；

[Using Search](http://docs.basho.com/riak/2.1.3/dev/using/search/)
[Search Details](http://docs.basho.com/riak/2.1.3/dev/advanced/search/)

#### Ring
Riak ring 由 160-bit 的整数空间构成；
该空间被等分为不同的分区，每一个分区对应一个 vnode ；
所有分区都驻留在实际物理服务器节点上；

[Clusters](http://docs.basho.com/riak/2.1.3/theory/concepts/Clusters/)
[Dynamo](http://docs.basho.com/riak/2.1.3/theory/dynamo/)
[Cluster Capacity Planning](http://docs.basho.com/riak/2.1.3/ops/building/planning/cluster/#Ring-Size-Number-of-Partitions)


#### Secondary Indexing (2i)
Riak 中的二级索引为开发者提供了基于一个或多个 value 标识（tag）和查询存储对象的能力；

[Using Secondary Indexes](http://docs.basho.com/riak/2.1.3/dev/using/2i/)
[Advanced Secondary Indexes](http://docs.basho.com/riak/2.1.3/dev/advanced/2i/)
[Repairing Indexes](http://docs.basho.com/riak/2.1.3/ops/running/recovery/repairing-indexes/)


#### Strong Consistency
尽管 Riak 是作为 [最终一致性](http://docs.basho.com/riak/2.1.3/theory/concepts/Eventual-Consistency/) 数据存储系统闻名的，但是从 Riak 2.0 开始，允许你针对部分或全部数据采用强一致性保证；
至此，Riak 既可以作为 CP(consistent plus partition-tolerant) 系统使用，也可以作为 AP(highly available plus partition-tolerant) 系统使用了；

[Strong Consistency Concept](http://docs.basho.com/riak/2.1.3/theory/concepts/strong-consistency/)
[Using Strong Consistency](http://docs.basho.com/riak/2.1.3/dev/advanced/strong-consistency/)


#### Value
Riak 经常被描述为 key/value 存储；
在 Riak 2.0 版本之前，其保存的 value 实际上为通过唯一 key 确定的 opaque BLOB (binary large objects) ；value 的内容可以是任何类型的数据，包括 string 类型，JSON 对象，文本文档等等；针对 value 的修改操作需要先从 Riak 中将其获取出来，再使用新的 value 值替代原来的 value 值保存回去；在这种情况下，针对 value 的操作只能是基本的 CRUD 操作；


[Riak Data Types](http://docs.basho.com/riak/2.1.3/theory/concepts/crdts/), added in version 2.0, are an important exception to this. While still considered values—because they are stored in bucket type/bucket/key locations, like anything in Riak—Riak Data Types are not BLOBs and are modified by Data Type-specific operations.
从 Riak 2.0 版本开始，增加了 [Riak Data Types](http://docs.basho.com/riak/2.1.3/theory/concepts/crdts/) 功能；
尽管仍称之为 value - 因为其被保存在 bucket type/bucket/key 位置，正如 Riak 中的其他部分一样 - 但 Riak Data Types 已不再是 BLOB 了，其已经支持数据类型相关的特定操作了；

[Keys and Objects](http://docs.basho.com/riak/2.1.3/theory/concepts/keys-and-values/)
[Developer Basics](http://docs.basho.com/riak/2.1.3/dev/using/basics/)
[Data Types](http://docs.basho.com/riak/2.1.3/theory/concepts/crdts/)
[Using Data Types](http://docs.basho.com/riak/2.1.3/dev/using/data-types/)


#### Vector Clock
Riak 基于向量时钟（或者称为 vclock）处理版本控制问题；
因为 Riak 集群中的任意节点均可以处理请求，但又不要求全部节点都要参与决策，那么就需要这样一种数据版本管控机制用于跟踪当前值；
当一个 value 被存储到 Riak 中时，会被使用一个向量时钟值对其进行标识（tag），同时建议数据的初始版本；
当 value 被更新时，客户端会提供修改对象的向量时钟值，以便基于该值反映出本次修改；
之后 Riak 就能够通过比较同一对象在不同版本下的向量时钟值，最终决定数据的某些属性状态；

[Vector clocks](http://docs.basho.com/riak/2.1.3/theory/concepts/Vector-Clocks/)

#### Vnode
vnode 也被称为虚拟节点，用于在 Riak ring 中对应某个分区；
vnode 负责协调针对特定分区的请求处理；

[Vnodes](http://docs.basho.com/riak/2.1.3/theory/concepts/vnodes/)
[Clusters](http://docs.basho.com/riak/2.1.3/theory/concepts/Clusters/)
[Dynamo](http://docs.basho.com/riak/2.1.3/theory/dynamo/)