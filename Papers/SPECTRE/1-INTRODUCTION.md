> **Source：** [https://eprint.iacr.org/2016/1159.pdf](https://eprint.iacr.org/2016/1159.pdf)  
> **TranStudy：** [https://github.com/DAGfans/TranStudy/new/master/Papers/SPECTRE](https://github.com/DAGfans/TranStudy/new/master/Papers/SPECTRE)

# 1.INTRODUCTION

Bitcoin is a novel cryptocurrency system, and an accompanying protocol, invented and deployed by Satoshi Nakamoto [13]. 
Transactions made in the currency are organized in a public ledger, the blockchain. 
Each block in the chain is a batch of transactions that were published by users of the currency. 
The blockchain contains consistent transactions only, as new blocks that extend it are required to maintain consistency.

比特币是一种新型的加密货币系统同时也是一个协议，由中本聪 [13]发明和部署。 
以货币形式进行的交易被组织在一个公共账本，即区块链。 
链中的每个块都是由货币用户发布的一批交易。 
区块链仅包含一致的交易，因为需要保持一致性才能扩展它的新块。

Unfortunately, Nakamoto Consensus has severe scalability limitations [5], [18], [14]: adjusting the protocol to support a high transaction throughput – by creating larger or more frequent blocks – requires stronger assumptions on the underlying network, hence smaller safety margins.

不幸的是，中本聪共识有严重的可扩展性限制[5],[18],[14]：通过调整协议来支持高交易吞吐量 - 通过创建更大或更频繁的块 - 需要对底层网络进行更强的假设，因此会导致更小的安全系数。

In this paper we propose a new protocol, SPECTRE, that achieves high scalability. 
Transactions in SPECTRE can be conﬁrmed within seconds, and the throughput can be improved by orders-of-magnitude over Bitcoin; 
it is limited by the network infrastructure and capacity only. 
The protocol thus alleviates the security-scalablility trade-off imposed by Nakamoto Consensus.

在本文中，我们提出了一种新的协议，即SPECTRE，它具有很高的可扩展性。 
SPECTRE中的交易可以在几秒钟内得到确认，吞吐量可以相较于比特币有数量级的提高; 
它仅受网络基础设施和容量的限制。 
因此该协议减轻了中本聪共识强加的安全可扩展性之间的取舍。

In SPECTRE, every block is counted and integrated into the ledger. 
Technically, SPECTRE generalizes Nakamoto’s blockchain into a direct acyclic graph – a block DAG. 
By maintaining a full DAG of blocks, SPECTRE can allow miners to create blocks concurrently and much more frequently. 
This design is intended to avoid the need for nodes to reconcile their different world views regarding the identity of a selected chain at the time of block creation.

在SPECTRE中，每个块都被包括并集成到账本中。 
从技术上讲，SPECTRE将中本聪的区块链泛化为有向无环图 - 区块DAG。 
（**译注：** 泛化这里指链式结构可以看作Block DAG的特例）
通过保持块的完整DAG，SPECTRE可以允许矿工并发并且高频地创建区块。 
这种设计旨在避免节点在块创建时协调其关于所选链的身份的不同世界观。
（**译注**：这句话指，所谓链式还是图式其实是因为看问题的角度不同，站在DAG的角度来看，链式也算是一种DAG）

Reasoning about the consensus properties of SPECTRE requires a new formal framework. 
Indeed, previous work that formalized the robustness of Nakamoto Consensus [7], [15] focused on robustness of blocks in the ledger. 
Extending this to the robustness of transactions in SPECTRE is not immediate, because all blocks are incorporated into the DAG, but individual transactions embedded in the DAG may still be rejected due to conﬂicts.

论证SPECTRE的共识特性需要一个新的形式化框架。 
事实上，之前形式化中本聪共识健壮性的工作专注在账本中的区块的健壮性上[7]，[15]。 
但这并不能立即扩展到SPECTRE中的交易的健壮性，因为所有的块都包含在DAG中，但嵌入在DAG中的单个交易可能因冲突而被拒绝。
**译注：** 冲突一般指的就是双花

Thus, this paper contains two contributions: 
(1) an inherently scalable protocol, SPECTRE; 
and (2) a formal framework for cryptocurrency payment protocols that do not necessarily use a chain of blocks to represent their ledger (in this respect we differ from previously proposed frameworks). 
We apply it on SPECTRE, and provide rigorous analysis of SPECTRE’s robustness properties.

因此，本文包含两个贡献：
（1）一个可扩展的协议，SPECTRE; 
（2）一个加密货币支付协议的形式化框架，它不一定要使用区块链表示账本（在这方面，我们不同于以前提出的框架）。 
我们将其应用于SPECTRE，并对SPECTER的健壮性进行严格分析。

The main technique behind SPECTRE is a voting algorithm regarding the order between each pair of blocks in the DAG. 
The voters are blocks (not miners); 
the vote of each block is interpreted algorithmically (and not provided interactively) according to its location within the DAG. 
We show that the majority’s aggregate vote becomes irreversible very fast, and we use this majority vote to extract a consistent set of transactions. 
Essentially, Bitcoin’s longest chain rule can be seen as a voting mechanism as well – each block adding one vote to every chain that contains it – the highest-scoring chain being also the longest one. 
However, Bitcoin’s selection of a “single winner chain” makes it inherently unscalable, as we demonstrate below.

SPECTRE背后的主要技术是关于DAG中对区块之间的顺序进行投票的算法。 
投票者是区块（不是矿工）; 
根据其在DAG内的位置通过算法表示每个块（并且不提供交互式）。 
我们表明，大多数的总体投票不可逆转的速度会很快，我们使用这个多数票来提取一组一致的交易。 
从本质上讲，比特币最长链规则也可以被看作是一种投票机制 - 每个区块给每一个包含它的链添加一票 - 最高得分的链也是最长的。 
但是，比特币选择的“单一赢家链”使得它本质上不可扩展，正如我们下面所示。

We note that there have been several recent works revolving around new protocols for public blockchain systems. 
These include Bitcoin-NG [6], Byzcoin [9], a work by Decker et. al. [4], Hybrid Consensus [16], Solidus [1], and recently Algorand [11]. 
We discuss these and other related works in Section 6.

我们注意到最近有几项工作围绕公有区块链系统的新协议。 
这些包括Bitcoin-NG [6]，Byzcoin [9]，Decker等人的工作[4]，混合共识（Hybrid Consensus）[16]，Solidus [1]和最近Algorand [11]。 
我们在第6节讨论这些和其他相关的工作。
