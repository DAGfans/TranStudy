
#### 1. INTRODUCTION
#### 1. 介绍 

##### A. The Bitcoin protocol  
##### A. 比特币协议

The Bitcoin protocol instructs miners how to create blocks of transactions. 
The body of a block contains new transactions published by users, a proof-of-work puzzle, and a pointer to one previous block. 
The latter rule implies that blocks naturally form in a tree structure. 
When creating his next block, the miner is instructed to reference the tip of the longest chain within the tree and ignore the rest of blocks (aka orphans).
Miners share and propagate a block immediately upon receiving or creating it, and reference the latest block in the chain they observe. 
The security of Bitcoin relies on honest nodes being sufficiently connected so that when one miner extends the chain with a new block, it propagates in time to all honest nodes before the next one is created.  
比特币协议指示矿工如何创建交易块。
块体包含用户发布的新交易，工作量证明难题和指向前一个块的指针。
后一条规则意味着块会自然形成树结构。
当创建下一个区块时，矿工会被指示引用树中最长链的末端，并忽略其余区块(又名孤块)。
矿工们在收到或创建区块后立即分享和传播该区块，并引用他们观察到的最新区块。
比特币的安全性依赖于诚实的节点被充分连接，以便当一个矿工用一个新块扩展链时，它会及时传播到所有诚实的节点，然后再创建下一个区块。

![fig 1](https://user-images.githubusercontent.com/22833166/37459465-2018d836-2883-11e8-9bc3-ee753150014d.jpg)

**Fig. 1:** An example of a block DAG G. 
Each block references all blocks to which its miner was aware at the time of its creation. 
The DAG terminology, applied to block H as an example, is as follows:
$past(H) = \left \{ Genesis,C,D,E \right \}$ – blocks which H references directly or indirectly, and which were provably created before H;
$future(H) =\left \{ {J,K,M} \right \}$– blocks which reference H directly or indirectly, and which were provably created after H;
$anticone(H) =\left \{ {B,F,I,L} \right \}$– the order between these blocks and H is ambiguous.
Deciding the order between H and blocks in $anticone(H)$ is the main challenge of a DAG protocol.
$tips(G) =\left \{ {J,L,M} \right \}$– leaf-blocks, namely, blocks with in-degree 0; 
these will be referenced in the header of the next block  
**图 1:** 块DAG G的一个例子。  
每个块都会引用矿工在创建时知道的所有块。  
DAG术语，以块H为例子，如下所示：  
$past(H) = \{Genesis,C,D,E\}$ - H直接或间接引用，并且很可能在H之前创建的块;  
$future(H) =\{J,K,M\}$ - 直接或间接引用H，并且很可能在H之后创建的块;  
$anticone(H) = \{B,F,I,L\}$ - 这些块与H之间的顺序是不明确的。
决定H和$anticone(H)$中块之间的顺序是一个DAG协议的主要挑战。  
$tips(G) = \{J,L,M\}$ - 叶块，即入度为0的块;
这些将在下一个块的头部中引用

In order to guarantee this property, the creation of blocks is regulated by the protocol to occur once every 10 minutes. 
As a result, Bitcoin suffers from a highly restrictive throughput in the order of 3-7 transactions per second (tps).  
为了保证这一特性，协议规定块的创建每10分钟进行一次。
因此，比特币的吞吐量高度受限, 大约每秒3-7次交易(tps)。

##### B. The PHANTOM protocol  
##### B. PHANTOM 协议

In this work we present PHANTOM, a protocol that enjoys a very large transaction throughput compared to Bitcoin. 
PHANTOM structures blocks in a Directed Acyclic Graph, a blockDAG.
Rather than extending a single chain, miners in PHANTOM are instructed to reference all blocks in the graph (that were not previously referenced, i.e., leaf-blocks). 
An example of a blockDAG, and the basic terminology, is provided in Figure 1 above.
The core challenge of a DAG protocol is how to order transactions embedded in it, so that in case of two (or more) conflicting transactions, we can accept the one that arrived first (according to the prescribed order) and reject the other(s).
To that end, PHANTOM relies on the interconnectivity of honest nodes (similarly to Bitcoin’s assumption in slow rates). 
Since cooperating PHANTOM miners propagate their blocks as soon as possible and reference all of their counterparts’ blocks, we should expect to see in the DAG a well-connected cluster of blocks. 
In contrast, blocks that were mined by non-cooperating nodes will appear as outliers and will easily be discerned. 
Indeed, deviation from PHANTOM’s mining protocol comes in the form of (i) withholding a new block for a while, or\and (ii) creating a new block that does not reference other blocks available at the time, both cases in which the new block can be recognized and penalized.
Following this intuition, together with the assumption that honest nodes hold a majority of the hashrate, we argue that the largest set of blocks with good inter-connectivity was mined by honest nodes, with high probability. 
Accordingly, given a blockDAG, we would want to solve the following optimization problem:  
在这项工作中，我们展示了PHANTOM，与比特币相比，该协议拥有非常大的交易吞吐量。
PHANTOM结构块在有向无环图，blockDAG中。
PHANTOM中的矿工不是扩展单链，而是引用图中所有的块(以前没有引用过，即叶块)。
图1中提供了一个blockDAG的例子和基本术语。
DAG协议的核心挑战是如何将嵌入其中的交易排序，以便在发生两次(或多次)冲突交易时，我们可以接受先到达的(按照规定的顺序)，并拒绝其他的。
为此，PHANTOM依赖诚实节点的互联性(类似于比特币的低速率假设)。
由于合作的PHANTOM矿工会尽快传播他们的区块，并引用其他矿工的区块，我们应该可以期待在DAG中看到一个连接良好的区块集群。
相反，由非协作节点挖掘的块将显示为异常值，并且很容易被识别。
实际上，偏离PHANTOM挖矿协议的形式是(i)扣留一个新区块，或者(ii)创建一个没有引用当时可用的其他区块的新区块，这两种情况下的区块都可以被识别并受到惩罚。
根据这个观察以及诚实节点占据大部分散列率(译注：可以理解为算力)的假设，我们认为具有良好互连性的最大的区块集合有很高的概率由诚实节点挖出。 
因此，给定一个blockDAG，我们想要解决下面的优化问题：

> **Maximum** $k$**-cluster SubDAG** ($MCS_k$)  
> **Input:** DAG $G= (C,E)$   
> **Output:** A subset $S^* \subset C$ of maximum size, s.t. $|anticone(B)\cap S^*| \leq k$ for all $B \in S^*$ .

Here, $anticone(B)$ is the set of blocks in the DAG which did not reference B(directly or indirectly via their predecessors) and were not referenced by B(directly or indirectly via B’s predecessors). 
The parameter k is related to an assumption that PHANTOM makes regarding the network’s propagation delay; 
this is explained in detail in Section 4, following the formal framework specified in Section 3.  
这里，$anticone(B)$是DAG中没有引用B(直接或通过它们的祖先间接)并且没有被B引用(直接或通过B的祖先间接)的块集合。
参数k与PHANTOM关于网络传播延迟的假设有关;
这在第4节中有详细解释，遵循第3节中规定的形式化框架。

In practice, the Maximum k-cluster SubDAG is NP hard (see problem [GT26] in [2]).
PHANTOM works therefore with a variant of this problem, using a greedy algorithm approach.
The algorithm will be given in Sections 2, and some interesting variants in Section 6.  
在实践中，最大k-集群SubDAG是NP问题(参见[2]中的问题[GT26])。
因此，PHANTOM使用贪婪算法的方法来解决这个问题的一个变种。
该算法将在第2节中给出，以及第6节中的一些有趣变体。

Once the set of honest and dishonest blocks are properly recognized by the protocol, we order the DAG in a way that favours the former set and penalizes the latter. 
Interestingly, the specific way in which we order honest blocks is of no importance to the security of the protocol—
any arbitrary rule which respects the topology will enjoy the robustness properties of PHANTOM, as we prove formally in Section 5.  
一旦诚实和不诚实的组合得到了协议的正确识别，我们就以有利于前者的方式对DAG排序，并惩罚后者。
有趣的是，我们排序诚实块的具体方式对于协议的安全性并不重要 - 
任何遵循拓扑结构的任意规则都将享有PHANTOM的健壮性，正如我们在第5节中形式化证明的那样。

However, the ordering rule does affect confirmation times. 
An arbitrary topological ordering might take a long while before converging, especially if an active visible attack is taking place.
Thus, vanilla PHANTOM does not guarantee fast convergence time in all cases. 
In Section 7 we elaborate on this, and discuss techniques and modifications that allow fast confirmation times.  
但是，排序规则确实会影响确认时间。
在收敛之前，任意的拓扑排序可能需要很长时间，特别是当发生活动的可见攻击时。
因此，在所有情况下，常规PHANTOM都不保证快速收敛时间。
在第7节中，我们详细阐述了这一点，并讨论了允许快速确认时间的技术和修改。


![fig 2](https://user-images.githubusercontent.com/22833166/37466668-513aa8f2-2899-11e8-96be-87aa97b56fe1.jpg)

**Fig. 2:** An example of the largest 3-cluster of blocks within a given DAG:A,B,C,D,F,G,I,J(coloured blue). 
It is easy to verify that each of these blue blocks has at most 3 blue blocks in its anticone, and (a bit less easy) that this is the largest set with this property. 
Setting PHANTOM’s inter-connectivity parameter with  $k= 3$ means that at most 4 blocks are assumed to be created within each unit of delay, so that typical anticone sizes should not exceed 3.
Blocks outside the largest 3-cluster,E,H,K(coloured red), belong to the attacker (w.h.p.). 
For instance, block E has 6 blue blocks in its anticone (B,C,D,F,G,I); 
these blocks didn’t reference E, presumably because E was withheld from their miners. 
Similarly, block K admits 6 blue blocks in its anticone (B,C,G,F,I,J); 
presumably, its malicious miner received already some blocks from(B,C,D,G), but violated the mining protocol by not referencing them.  
**图2：** 给定DAG中最大的 _3-集群_ 的示例：A，B，C，D，F，G，I，J(蓝色)。
很容易验证这些蓝色块中每个蓝色块在其反锥体中最多有3个蓝色块，并且(稍微不容易)该集合是具有该属性的最大集合。
设置PHANTOM的互连参数k = 3意味着最多4个块被假定为在每个延迟单元内创建，因此典型的反锥体尺寸不应该超过3。
最大的 _3-集群_ 以外的块E，H，K(红色)属于攻击者(很大可能)。
例如，区块E的反锥体有6个蓝色块(B，C，D，F，G，I)。
这些区块没有引用E，大概是因为E被矿工扣留。
类似地，区块K接受在其反锥体(B，C，G，F，I，J)中有6个蓝色方块;
据推测，其恶意矿工已经收到了来自(B，C，D，G)的一些块，但没有引用它们而违反了挖矿协议。

##### C. Related work  
##### C. 相关工作

Many suggestions to improve Bitcoin’s scalability have been proposed in recent years. 
These proposals fall into two categories, on-chain scaling and off-chain scaling. 
Roughly speaking, the former includes protocols where all valid transactions are those that appear – as in Bitcoin – inside blocks that are organized in some data structure (aka “the ledger”).  
近年来提出了许多改善比特币可扩展性的建议。
这些建议分为两类，即链上扩容和链下扩容。
粗略地说，前者包括的协议要求所有有效的交易都被组织在被称为账本的数据结构中，如比特币，交易被组织在区块中。

**On-chain scaling.** The protocols in this category may differ e.g. in how fast blocks are created, how blocks are organized in the ledger (a chain, a tree, a DAG, etc.), which transactions in the ledger are considered valid, and more. 
PHANTOM belongs to this line of works. 
Previous works in this family of protocols includes GHOST [9], where a main chain of blocks is chosen according to a greedy algorithm and not through the longest chain rule; 
Inclusive [5], where any chain-selection rule is extended to an ordered DAG and transactions off the main chain are added in a consistent manner; 
Bitcoin NG [1], where the ledger consists of slow key blocks (containing no transactions) and fast micro blocks that contain transactions. 
The sole purpose of key blocks in Bitcoin NG is to define the miner that is eligible to create micro blocks in that epoch and confirm thus transactions at a high rate.   
**链上扩容.** 该类协议分为很多种，例如 在创建块的速度方面，在账本中如何组织块(链，树，DAG等)，账本中的哪些交易被认为是有效的等等。
PHANTOM属于这一类。
在这个协议族中，以前的作品包括GHOST [9]，其根据贪婪算法选择块的主链，而不是通过最长链规则;
Inclusive [5]，其中任何链选择规则被扩展到一个有序的DAG，并且非主链上的交易以一致的方式被添加;
Bitcoin NG [1]，其中帐本由慢速关键块(不包含交易)和包含交易的快速微块组成。
Bitcoin NG中关键块的唯一目的是确定有资格在该时期创建微块的矿工，并确认交易的速度很快。  

GHOST is still susceptible to some attacks, one of which was described in [3]. 
The DAG in Inclusive adds throughput but not security to the main chain, hence suffers from the same limitations as the underlying main chain selection rule. 
Key blocks in Bitcoin NG are still generated slowly, thus confirmation times remain high.  
GHOST仍然容易受到一些攻击，其中之一在[3]中有所描述。
Inclusive中的DAG增加了主链的吞吐量但没有增加安全性，因此会受到与底层主链选择规则相同的限制。
Bitcoin NG中关键的块仍然会缓慢产生，因此确认时间仍然很长。 

Our work is most similar to the SPECTRE protocol [8]. 
SPECTRE enjoys both high throughput and fast confirmation times. 
It uses the structure of the DAG as representing an abstract vote regarding the order between each pair of blocks. 
One caveat of SPECTRE is that the output of this pairwise ordering may not be extendable to a full linear ordering, due to possible Condorcet cycles. 
PHANTOM solves this issue and provides a linear ordering over the blocks of the DAG.
As such, PHANTOM can support consensus regarding any general computation, also known as Smart Contracts, which SPECTRE cannot. 
Indeed, in order for a computation or contract to be processed correctly and consistently, the full order of events in the ledger is usually required, and particularly the order of inputs to the contract.^1 
PHANTOM’s linear ordering does not come without cost—confirmation times are mush slower than those in SPECTRE. 
In Section 7 we describe how the same system can simultaneously enjoy the best of both protocols.  
我们的工作与SPECTRE协议最相似[8]。
SPECTRE拥有高吞吐量和快速确认时间。
它使用DAG的结构来表示关于每对块之间的顺序的抽象投票。
SPECTRE值得注意的是，由于可能的Condorcet循环(译注：比如说A>B, B>C 但是A<C)，这种成对排序的输出可能无法扩展到全线性排序。
PHANTOM解决了这个问题，并提供了DAG块的线性排序。
因此，PHANTOM可以支持关于任何一般计算的共识，也称为智能合约，而SPECTRE不能。
事实上，为了正确和一致地处理计算或合约，通常需要账本中事件的完整顺序，特别是合约输入的顺序。^1 
PHANTOM的线性排序不会没有成本 - 确认时间比SPECTRE中的要慢。
在第7节中，我们描述了同一个系统如何同时享有这两种协议中的优点。

**Off-chain scaling.** Another totally different approach keeps block creations infrequent and their sizes small (so that propagation delay remains negligible), yet this slow chain is not used for recording the entire economic activity. 
Instead, most of the transactions occur outside the chain, with better scalability, and the chain itself is used for the purpose of resolving conflicts or settling transactions. 
One example is Hybrid Consensus [6], improving over [4], which uses the chain to select a rotating committee of nodes which in turn run a classic consensus protocol to confirm transactions in the corresponding epoch. 
Another well known proposed solution in the same category is the Lightning Network [7] (LN), where transactions are processed off-chain over over a network of micro payment channels, and the blockchain is used only for settlement of these channels.  
**链下扩容.** 另一种完全不同的方法是使块创建频度降低并且让尺寸变小(因此传播延迟可以忽略不计)，所以这个慢速链不会记录整个经济活动。
相反，大多数交易发生在链外，具有更好的可扩展性，链本身用于解决冲突或结算交易。
一个例子是混合共识[6]，对[4]的改进，它使用链选择一个循环的节点委员会，然后运行经典的共识协议来确认相应时期的交易。
另一个众所周知的同类解决方案是闪电网络 [7]（LN），其中交易是通过微支付通道网络进行离线处理的，区块链仅用于结算这些通道。

Our work is orthogonal and complementary to these solutions, and can enhance their operation by orders-of-magnitude. 
For instance, when the DAG is used to serve channel-settlement transactions of LN, it allows for a much cheaper access (due to larger supply of blocks and capacity) and much faster processing than if the LN were operating over a chain.  
我们的工作与这些解决方案是不相关的和互补的，并且可以按数量级增强起运行。  
例如，当DAG用于闪电网络的通道结算交易时，它允许更低成本的访问(由于有更大的块数量和容量的供应)以及比闪电网络运行在链上快得多的处理速度。

(^1) Contracts that do not require such a strict ordering can indeed be served under SPECTRE as well.  
(^1) 不需要这种严格排序的合约也可以在SPECTRE下提供。
