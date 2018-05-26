原文：[https://blog.daglabs.com/an-introduction-to-the-blockdag-paradigm-50027f44facb](https://blog.daglabs.com/an-introduction-to-the-blockdag-paradigm-50027f44facb)

#### An Introduction to the BlockDAG Paradigm

#### BlockDAG 范式入门

*Contrary to popular belief, using a DAG (directed acyclic graph) as a distributed ledger is not about removing proof-of-work mining, blocks, or transaction fees. It is about leveraging the structural properties of DAGs to potentially solve blockchain’s orphan rate problem. The ability of a DAG to withstand this problem and thus improve on scalability is contingent on the additional rules implemented to deal with transaction consistency, and any other design choices made.*

*与大众的普遍观点相反，将 DAG（有向无环图）用作分布式账本并不会消除工作量证明式的挖矿，也不会消除区块和交易手续费。它的目的是利用 DAG 的结构特性帮助解决区块链的孤儿率问题。DAG 解决这个问题并且改进可扩展性的能力取决于为了处理交易一致性而实现的额外规则，以及其它设计上所作的选择。*

##### Directed acyclic graphs

##### 有向无环图

A DAG is not a novel concept or technology, and it is definitely not a consensus mechanism; it is purely a mathematical structure originating from centuries ago. Technically, a DAG is a graph with directed edges and no cycles (i.e., there is no path from a vertex back to itself).

DAG 并不是一个新概念或是技术，并且它当然也不是一个共识机制；它纯粹是起源于几个世纪前的一种数学结构。在技术上，DAG 是一种包含有向边但不含环（即没有一条可以从一个顶点出发然后又回到它自己的路径）的图。

![DAG](https://cdn-images-1.medium.com/max/1600/1*aYmhytXcO6yxNpatmPwIIA.png)

In the context of distributed ledgers, a blockDAG is a DAG whose vertices represent blocks and whose edges represent references from blocks to their predecessors. Evidently, in a blockDAG, blocks may have several predecessors instead of just one; this will be described in more detail below. First, let us recall the orphan rate problem.

在分布式账本的环境中，blockDAG 一种有特殊含义的 DAG。在这种 DAG 中，顶点代表区块，边代表区块对它们父辈的引用。显然，在 blockDAG 中，区块可能有多个父辈，而不是只有一个；下文会对此进行详细描述。首先，让我们回顾一下孤儿率问题。

##### Blockchain’s orphan rate problem

##### 区块链的孤儿率问题

There are many barriers to blockchain scalability, including processing speeds, disk I/O, RAM, bandwidth, and syncing new nodes. While these limitations can be addressed with improved hardware and technology, a major barrier exists on the protocol level: the orphan rate. Orphans are blocks that are created outside the longest chain due to unavoidable network propagation delays.

有许多因素制约着区块链的可扩展性，包括处理速度、磁盘 I/O、RAM、带宽，以及新节点的同步（译注：这里应该是指网络扩容的时候发现新节点）。虽说可以通过改善硬件和技术来突破这些限制，但一个最主要的瓶颈是在协议层面：孤儿率。孤儿是指由于不可避免的网络延迟而在最长链以外创建的区块。

Accelerating block creation and/or increasing block sizes increases the orphan rate: by the time a new block propagates throughout the network, other new blocks which do not reference it are likely to be created. It is well established that a high orphan rate amounts to compromised security; the more honest blocks that end up outside the longest chain due to spontaneous forks, the less secure is the chain. ^1

加快区块创建速度以及／或者增大区块大小会增大孤儿率：在一个新区块传播遍网络之前，很可能会有其它新的不引用它的区块被创建出来。孤儿率增大会导致安全性降低，这是公认的；如果有越多的诚实区块因为
