原文：[https://blog.daglabs.com/an-introduction-to-the-blockdag-paradigm-50027f44facb](https://blog.daglabs.com/an-introduction-to-the-blockdag-paradigm-50027f44facb)

## An Introduction to the BlockDAG Paradigm

## BlockDAG 范式入门

*Contrary to popular belief, using a DAG (directed acyclic graph) as a distributed ledger is not about removing proof-of-work mining, blocks, or transaction fees. It is about leveraging the structural properties of DAGs to potentially solve blockchain’s orphan rate problem. The ability of a DAG to withstand this problem and thus improve on scalability is contingent on the additional rules implemented to deal with transaction consistency, and any other design choices made.*

*与大众的普遍观点相反，将 DAG（有向无环图）用作分布式账本并不会消除工作量证明式的挖矿，也不会消除区块和交易手续费。它的目的是利用 DAG 的结构特性帮助解决区块链的孤块率问题。DAG 解决这个问题并且改进可扩展性的能力取决于为了处理交易一致性而实现的额外规则，以及其它设计上所作的选择。*

### Directed acyclic graphs

### 有向无环图

A DAG is not a novel concept or technology, and it is definitely not a consensus mechanism; it is purely a mathematical structure originating from centuries ago. Technically, a DAG is a graph with directed edges and no cycles (i.e., there is no path from a vertex back to itself).

DAG 并不是一个新概念或是技术，并且它当然也不是一个共识机制；它纯粹是起源于几个世纪前的一种数学结构。在技术上，DAG 是一种包含有向边但不含环（即没有一条可以从一个顶点出发然后又回到它自己的路径）的图。

![DAG](https://cdn-images-1.medium.com/max/1600/1*aYmhytXcO6yxNpatmPwIIA.png)

In the context of distributed ledgers, a blockDAG is a DAG whose vertices represent blocks and whose edges represent references from blocks to their predecessors. Evidently, in a blockDAG, blocks may have several predecessors instead of just one; this will be described in more detail below. First, let us recall the orphan rate problem.

在分布式账本的环境中，blockDAG 一种有特殊含义的 DAG。在这种 DAG 中，顶点代表区块，边代表区块对它们父辈的引用。显然，在 blockDAG 中，区块可能有多个父辈，而不是只有一个；下文会对此进行详细描述。首先，让我们回顾一下孤块率问题。

### Blockchain’s orphan rate problem

### 区块链的孤块率问题

There are many barriers to blockchain scalability, including processing speeds, disk I/O, RAM, bandwidth, and syncing new nodes. While these limitations can be addressed with improved hardware and technology, a major barrier exists on the protocol level: the orphan rate. Orphans are blocks that are created outside the longest chain due to unavoidable network propagation delays.

有许多因素制约着区块链的可扩展性，包括处理速度、磁盘 I/O、RAM、带宽，以及新节点的同步（译注：即新节点刚加入网络时将整条区块链下载同步到自己本地的过程）。虽说可以通过改善硬件和技术来突破这些限制，但一个最主要的瓶颈是在协议层面：孤块率。孤块是指由于不可避免的网络延迟而在最长链以外创建的区块。

Accelerating block creation and/or increasing block sizes increases the orphan rate: by the time a new block propagates throughout the network, other new blocks which do not reference it are likely to be created. It is well established that a high orphan rate amounts to compromised security; the more honest blocks that end up outside the longest chain due to spontaneous forks, the less secure is the chain. ^1

加快区块创建速度以及／或者增大区块大小会增大孤块率：在一个新区块传播遍网络之前，很可能会有其它新的不引用它的区块被创建出来。孤块率增大会导致安全性降低，这是公认的；如果有越多的诚实区块因为自然分叉的原因导致自己不在最长链中，那么这条链就越不安全。^1 （译注：这里的自然分叉是指由于协议本身的特性和网络延迟导致的分叉，而非攻击导致的分叉。在最长链规则下，分叉越多，攻击者为了使自己成为最长链而需要的算力就越小，因此最长链就越不安全。比如在有三条分叉的情况下，攻击者只需超过全网三分之一的算力即可攻占最长链。因此从协议本身的角度来讲，自然分叉越少越好。）

Blockchain protocols typically impose a maximum block size and constant block creation rate to accommodate the network propagation and minimize orphans. This artificial limit of transaction throughput and lower bound on latency (in Bitcoin’s case, to 3–7 transactions per second and tens of minutes confirmation times) is a hard pill for blockchains to swallow — while it impedes on-chain scaling, it guarantees that spontaneous forks and orphans are rare and, therefore, that the main chain is secure. DAG protocols, however, may deal with orphans in other ways.

区块链协议一般会规定一个最大区块大小和一个常数的出块率从而应对网络延时并且将孤块的数量降到最低。这种对交易吞吐量和等待时间下限的人工限制（在比特币的情况下，这个限制是每秒3至7个交易和数十分钟的确认时间）对区块链来说是一副苦药丸——它阻止了链上扩容，但保证了自然分叉和孤块极少出现，因此主链是安全的。然而 DAG 可以通过其它方式处理孤块问题。

### The blockDAG paradigm

### blockDAG 范式

The notion of a fork is organically absorbed in the DAG framework, so it seems worthwhile to consider if a DAG could do better than the chain/linked list structure of blockchains. Accordingly, with Satoshi’s proof-of-work system as the starting point, we need to make one change to the mining protocol in order to yield a blockDAG: blocks may reference multiple predecessors instead of a single parent. A canonical way to extend the ledger is to have blocks reference all tips of the graph (that their miners observe locally) instead of referencing the tip of the single longest chain, as in Satoshi’s original protocol.

DAG 框架有机地吸收了分叉这一概念，所以看上去 DAG 是否可以比链式／链表结构的区块链做得更好这件事情是值得我们考虑的。于是，以中本聪的工作量证明系统为基础，为了生成一个 blockDAG，我们需要对挖矿协议做出一个改变：区块可以引用多个父辈，而非一个单一的父亲。一种典型的扩展账本的方式是让区块引用（产生区块的挖矿者在本地能看到的）图中的所有末端，而非依照中本聪的原始协议只引用最长链的末端。

![blockDAG](https://cdn-images-1.medium.com/max/2000/1*YJgJTzHlnrXrDU_ddsWtAA.png)

In a canonical blockDAG ledger, new blocks reference all tips of the graph (blocks that have not yet been referenced) that their miners see locally. As in a blockchain, blocks are published immediately.

在一个典型的 blockDAG 账本中，新区块引用它们的挖矿者在本地看到的图中的所有末端（即还未被引用的区块）。和在区块链中一样，区块会被立即发布。

However, unlike a blockchain which, by construction, preserves consistency (every block in the chain adds transactions that are consistent with its predecessors in the chain), a blockDAG incorporates blocks from different “branches” and so may contain many conflicting transactions. Because of this, a DAG, or blockDAG, cannot be considered a “solution” or “novel approach” or “new protocol” in and of itself. Instead, a blockDAG is a framework for devising consensus protocols that may (or may not) be as secure as and more scalable than chain-based protocols.²

然而，和区块链不同的是，区块链在构建时会一直维护着一致性（链中的每个区块添加的交易都与链中的父辈一致），而 blockDAG 包含了来自不同“分支”的区块，所以可能包含许多冲突交易。因此，DAG 或是 blockDAG 本身并不能被认为是一个“解决方案”或是“新方法“或是”新协议“。blockDAG 只是一个用来设计比链式协议扩展性更高的共识协议的框架，而所设计出的协议可能（也可能不）具有链式协议同等的安全性。

We therefore need a method to recover consistency; in other words, a blockDAG system requires replacing Satoshi’s longest chain rule with a new consensus protocol.

因此我们需要一种方法来恢复一致性；换句话说，一个 blockDAG 系统需要用一个新的共识协议来取代中本聪的最长链规则。

#### Consensus via ordering

#### 通过排序达成共识

Observe that if a distributed system achieves consensus on the order of all events in it, then we can easily extend this agreement to achieve consensus on the state — we simply iterate over all transactions according to the agreed order, and accept each transaction that is consistent with those accepted so far. This method preserves consistency by construction.

如果一个分布式系统对系统中所有事件的顺序达成共识，我们就可以很容易地将这个共识扩展到状态的共识——只需要简单地按照达成共识的顺序遍历所有交易，并且接受那些与已经被接受的交易一致的交易。这种方法在构建区块链的过程中就一致维护着一致性。

译注：注意作者在这里用的词是 accept，即“接受”。这个词容易与 confirm 即“确认”混淆。交易被接受不代表交易被确认。即便系统在某个时刻对交易排序达成共识，随着时间的推移，新交易的加入，这个排序可能会发生改变。系统可能会否定旧排序，转而对新排序达成共识。只有当排序不会随着时间推移而变化，交易才算是被“确认”。

We are left with the task of defining an ordering protocol on all events in the system — in our context, on all blocks in the DAG — in a way that will be agreed upon, eventually, by all nodes.

我们现在要做的就是为系统中的所有事件——在我们的上下文中，就是 DAG 中的所有区块——定义一个排序协议。所有节点都会遵从这个协议从而在区块顺序上最重达成共识。

The natural topology of a DAG already induces a partial ordering over the blocks: if there is a path from block X to block Y in the DAG, then block Y was provably created before block X and so should precede X in the global order, and vice versa. Thus, we only need to define an order over blocks created in parallel, i.e. any set of blocks such that there is no path that connects them.

DAG 的自然拓扑本身就已经为区块引入了偏序排序：若在 DAG 中有一条路径从区块 X 到区块 Y，即可证明区块 Y 是在区块 X 之前被创建的，因此在全局排序中应该排在 X 前面，反之亦然。因此，我们只需要为并行创建的区块，也就是相互之间没有路径相连的区块所组成的集定义一种顺序。

This paradigm began with the blockDAG-based protocols developed out of the Hebrew University ([Inclusive](http://www.cs.huji.ac.il/~yoni_sompo/pubs/15/inclusive_full.pdf), [SPECTRE](http://www.cs.huji.ac.il/~yoni_sompo/pubs/17/SPECTRE.pdf), and [PHANTOM](https://eprint.iacr.org/2018/104.pdf)); these protocols each define an algorithm that outputs an order over the DAG’s blocks, iterates the DAG by that order, and eliminates transactions that conflict with previous ones. (Actually, SPECTRE does something slightly weaker, but that’s a topic for a separate blog post.)

这种范式起源于耶路撒冷希伯来大学开发的基于 blockDAG 的协议（[Inclusive](http://www.cs.huji.ac.il/~yoni_sompo/pubs/15/inclusive_full.pdf)、[SPECTRE](http://www.cs.huji.ac.il/~yoni_sompo/pubs/17/SPECTRE.pdf)，和 [PHANTOM](https://eprint.iacr.org/2018/104.pdf)）；这些协议分别定义了自己的算法，每个算法都对 DAG 中的区块进行排序，并按顺序遍历 DAG，消除与已经被遍历过的交易相冲突的交易。（实际上，SPECTRE 做的事情弱一些，不过这需要另写一篇单独的博文来论述。）

#### Advantages of blockDAGs

#### blockDAG 的优势

BlockDAG protocols such as SPECTRE and PHANTOM circumvent the problems associated with high orphan rates. This comes with many advantages:

SPECTRE 和 PHANTOM 这样的 BlockDAG 协议规避了与高孤块率相关联的问题。这就带来了很多好处：

1\. It allows for confirmation times on the order of seconds, at least when there are visible double-spends and conflicts

1\. 确认时间可以以秒计算，至少是在有可见的双花和冲突交易时

2\. It allows for a large transaction throughput, limited only by the network backbone and endpoints’ capacity; as a derivative, it implies low fees

2\. 交易吞吐量可以变得很大，只受限于网络条件和终端能力；这样衍生出的好处是手续费会降低

3\. It contributes to mining decentralization by allowing for roughly 100,000 blocks per day, which reduces the incentive to join a mining pool

3\. 因为每天可以创建大约 100,000 个区块，所以矿工加入矿池的动机也会变弱，这进一步促成了挖矿去中心化

4\. It avoids the risk of orphaning, which comes with many additional benefits (such as Layer Two compatibility)

4\. 规避了孤块的风险，从而带来很多额外好处（比如与第二层兼容）（译注：第二层是指侧链或链下的解决方案，即在主链之上，新建一层辅助的平台和协议。比如[闪电网络](https://lightning.network/)，具体可参考《[区块链水平扩展：第二层网络](https://www.jianshu.com/p/8b5327c6d273)》。）

5\. It eliminates selfish mining by rewarding all blocks without discriminating between on-chain and off-chain blocks

5\. 通过奖励所有区块，消除链上和链下区块的差别，从而消除自私的挖矿行为

We will expand on each of these points and how SPECTRE and PHANTOM achieve them in future blog posts.

我们会在未来的博客文章里对上面的每一点进行具体论述，并讲述 SPECTRE 和 PHANTOM 是如何做到上面这几点的。

#### BlockDAGs vs. blockless DAGs

#### BlockDAG vs. 无区块 DAG

Almost every single DAG-based cryptocurrency on the market (IOTA, Byteball, Nano, etc.) has deviated from Satoshi’s blockchain paradigm, not only by using the DAG structure, but also in economic design: some have relegated mining to their users, some have eliminated proof-of-work mining altogether, many have no transaction fees, and practically all have no blocks, chaining together individual transactions. These design decisions may work in a DAG system, but they are characteristics independent of DAGs. In fact, these projects’ use of a DAG is probably their least defining characteristic.

市场上几乎每种基于 DAG 的加密货币（IOTA、字节雪球、Nano，等等）都偏离了中本聪的区块链范式。这种偏离不仅体现在使用 DAG 结构上，还体现在经济学层面的设计上：有的将挖矿委托给用户，有的完全去除了工作量证明挖矿，许多货币没有交易手续费，并且实际上都没有区块——它们将一个个交易直接连在一起。虽然这些设计决策可以用在 DAG 系统中，但它们的特性和 DAG 完全无关。实际上，这些项目可能只用到了 DAG 特性的很小一部分。

The rising popularity of these DAG systems has heavily influenced the community’s perception of DAGs. On the one hand, there are the fervent supporters who tout DAG technology as “blockchain 3.0”, and on the other, the skeptics who dismiss it as vaporware. Almost all, however, praise or criticize the economic design choices of existing DAG protocols, which have nothing to do with DAGs.

这些 DAG 系统的兴起很大程度上影响了社区对 DAG 的看法。一方面，狂热支持派将 DAG 技术吹捧为“区块链3.0”；另一方面，怀疑论者对其不屑一顾。可是几乎所有人赞扬或批评的都是现存 DAG 协议在经济层面的设计决策，而这和 DAG 根本毫无关系。

For instance, in a recent Q&A, renowned blockchain expert Andreas Antonopoulos described DAGs as “an alternative consensus mechanism” opposed to proof-of-work. “If you don’t have decentralized proof-of-work mining, you don’t really need blocks,” Antonopoulos says. “You can just chain transactions together, which is the basis of directed acyclic graphs.”

比如，在最近的一个问答中，著名比特币专家 Andreas Antonopoulos 将 DAG 描述为和工作量证明相对的“另一种共识机制”。“如果你没有将工作量证明挖矿去中心化，你其实并不需要区块。” Antonopoulos 说道，“你只需要直接将交易连起来，这就是有向无环图的基础。”

[点击查看 Andreas Antonopoulos 的问答视频](https://youtu.be/lfgMnbb5JeM)

But, as explained above, DAGs are not about replacing proof-of-work or blocks. DAGs are merely a mathematical structure that happen to be used by several projects that deviate from Satoshi’s proof-of-work system. In contrast, blockDAGs are the applications of DAGs to a Nakamoto-based system (in particular, with proof-of-work), only redesigning the data structure and consensus layer.

但是上文已经解释过，DAG 并不是要取代工作量证明或区块。DAG 只是一种数学结构。只是这种数学结构恰好被一些偏离了中本聪的工作量证明系统的项目所使用。与此相反，blockDAG 是要将 DAG 应用在基于中本聪的系统中（并且特别要强调的是，blockDAG 使用工作量证明），只是将数据结构和共识层重新设计了一遍。

Bram Cohen of BitTorrent and Chia hits closer to the mark with this scathing tweet:

比特流（BitTorrent）和 Chia 的创始人布拉姆·科恩（Bram Cohen）在推特上发表的尖锐评论更接近本质：
