> Source：https://eprint.iacr.org/2018/104.pdf  
> TranStudy: https://github.com/DAGfans/TranStudy/edit/master/Papers/PHANTOM%20-%20A%20Scalable%20BlockDAG%20protocol.md

# PHANTOM: A Scalable BlockDAG protocol
# PHANTOM: 可扩展的 BlockDAG 协议

### Yonatan Sompolinsky and Aviv Zohar

### School of Engineering and Computer Science,
### 计算机科学与工程学院

### The Hebrew University of Jerusalem, Israel
### 耶路撒冷希伯来大学，以色列

### {yoni sompo,avivz}@cs.huji.ac.il

## Abstract

> In 2008 Satoshi Nakamoto invented the basis for what would come to be known as blockchain technology. 
The core concept of this system is an open and anonymous network of nodes, or miners, which together maintain a public ledger of transactions. 
The ledger takes the form of a chain of blocks, the blockchain, where each block is a batch of new transactions collected from users. 
One primary problem with Satoshi’s blockchain is its highly limited scalability. 
The security of Satoshi’s longest chain rule, more generally known as the Bitcoin protocol, requires that all honest nodes be aware of each other’s blocks in real time. 
To this end, the throughput is artificially suppressed so that each block fully propagates before the next one is created, and that no “orphan blocks” that fork the chain be created spontaneously.
In this paper we present PHANTOM, a protocol for transaction confirmation that is secure under any throughput that the network can support. 
PHANTOM thus does not suffer from the security-scalability tradeoff which Satoshi’s protocol suffers from. 
PHANTOM utilizes a Directed Acyclic Graph of blocks, aka blockDAG, a generalization of Satoshi’s chain which better suits a setup of fast or large blocks. 
PHANTOM uses a greedy algorithm on the blockDAG to distinguish between blocks mined properly by honest nodes and those mined by non-cooperating nodes that deviated from the DAG mining protocol. 
Using this distinction, PHANTOM provides a full order on the blockDAG in a way that is eventually agreed upon by all honest nodes.

## 摘要

> 2008年，中本聪发明了被称为区块链技术的基础。
这个系统的核心概念是一个开放和匿名的节点或矿工网络，
它们共同维护着交易的公共账本。
账本采取区块的链式形式，即区块链，每个区块是从用户收集的一批新交易。
中本聪式区块链的一个主要问题是其可扩展性非常有限。
中本聪的最长链规则(通常称为比特币协议)的安全性，要求所有诚实的节点实时地了解彼此的块。
为此，吞吐量被人为地抑制，使得每个块在下一个块被创建之前可以完全传播，并且没有任何分叉产生的“孤块”可以自发创建。
在本文中，我们介绍PHANTOM，用于交易确认的协议，在网络可支持的任何吞吐量下都是安全的。
因此，PHANTOM没有中本聪协议所面临的要在安全和可扩展性之间权衡的问题。
PHANTOM利用有向无环图区块，又称blockDAG，一个更适合快速或大区块配置下的广义上的中本聪式区块链。
PHANTOM在blockDAG上使用贪婪算法来区分诚实节点正确挖出的区块和偏离DAG采矿协议的非协作节点挖出的区块。
利用这个区别，PHANTOM以最终由所有诚实节点同意的方式在blockDAG上提供全序。

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
past(H) = {Genesis,C,D,E} – blocks which H references directly or indirectly, and which were provably created before H;
future(H) ={J,K,M}– blocks which reference H directly or indirectly, and which were provably created after H;
anticone(H) = {B,F,I,L}– the order between these blocks and H is ambiguous.
Deciding the order between H and blocks in anticone(H) is the main challenge of a DAG protocol.
tips(G) = {J,L,M}– leaf-blocks, namely, blocks with in-degree 0; 
these will be referenced in the header of the next block  

**图 1:** 块DAG G的一个例子。  
每个块都会引用矿工在创建时知道的所有块。  
DAG术语，以块H为例子，如下所示：  
past(H) = {Genesis,C,D,E} - H直接或间接引用，并且很可能在H之前创建的块;  
future(H) ={J,K,M} - 直接或间接引用H，并且很可能在H之后创建的块;  
anticone(H) = {B,F,I,L} - 这些块与H之间的顺序是不明确的。
决定H和anticone(H)中块之间的顺序是一个DAG协议的主要挑战。  
tips(G) = {J,L,M - 叶块，即入度为0的块;
这些将在下一个块的头部中引用

译注：cone在英文里的意思是锥体，因此anticone直译过来就是“反锥体“。实际上anticone(H)就是既不是H的父辈，也不是H的后辈，并且也不是H本身的块组成的集合。当图很庞大的时候，H的父辈集和后辈集的形状分别就像一个以H为顶端的锥体（cone）。作者可能是基于这一点将父辈、后辈及H以外的区块集命名为反锥体（anticone）。

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

> **Maximum** _k_**-cluster SubDAG** (_MCS<sub>k</sub>_)  
> **Input:** DAG _G= (C,E)_   
> **Output:** A subset _S<sup>\*</sup> ⊂ C_ of maximum size, s.t. _|anticone(B)∩S<sup>*</sup>|≤k_ for all _B∈S<sup>\*</sup>_ .

Here, anticone(B) is the set of blocks in the DAG which did not reference B(directly or indirectly via their predecessors) and were not referenced by B(directly or indirectly via B’s predecessors). 
The parameter k is related to an assumption that PHANTOM makes regarding the network’s propagation delay; 
this is explained in detail in Section 4, following the formal framework specified in Section 3.  
这里，anticone(B)是DAG中没有引用B(直接或通过它们的祖先间接)并且没有被B引用(直接或通过B的祖先间接)的块集合。
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
Setting PHANTOM’s inter-connectivity parameter with k= 3 means that at most 4 blocks are assumed to be created within each unit of delay, so that typical anticone sizes should not exceed 3.
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

#### 2. THE PHANTOM PROTOCOL
#### 2. PHANTOM 协议

In this section we describe the operation of the PHANTOM protocol. 
PHANTOM consists of the following three-step procedure:  
在本节中，我们将介绍PHANTOM协议的操作。
PHANTOM包含以下三个步骤：

> 1) Using the structure of the DAG, we recognize in it a cluster of well-connected blocks; 
with high probability, blocks that were mined honestly belong to this cluster and vice versa.
> 2) We extend the DAG’s natural partial ordering to a full topological ordering in a way that favours blocks inside the selected cluster and penalizes those outside it.
> 3) The order over blocks induces an order over transactions; 
transactions in the same block are ordered according to the order of their appearance in it. 
We iterate over all transactions in this order, and accept each one that is consistent (according to the underlying Consistency notion) with those approved so far.
The Consistency notion used in the last step depends on the specific application under consideration. 
For instance, with regards to the Payments application, a transaction is consistent with the history only if all of its inputs have been approved and no double spending transaction has been approved before. 
Our work is agnostic to the definition of the Consistency rule. 
The contribution of PHANTOM is its implementation of the first two steps described above, which we now turn to describe.

> 1) 使用DAG的结构，我们在其中识别出连接良好的区块集群;
很有可能，被诚实挖出的块属于这个集群，反之亦然。
> 2) 我们用某种方式将DAG天然的偏序扩展为全序，该方式奖励集群内的块，并对集群外的块进行惩罚。
> 3) 块的顺序也会引发交易的顺序(译注：应该是指，如果块A的顺序大于块B，则块A内的所有交易的顺序都大于块B内的交易);
同一块的交易根据它们在内部出现的顺序排序.
我们按照此顺序遍历所有交易，并接受每一笔与迄今已确认的交易一致(根据基本的一致性概念)的交易。
最后一步中使用的一致性概念取决于所考虑的具体应用。
例如，关于“支付”应用程序，只有在所有输入都被确认并且之前没有确认过双花交易的情况下，交易才符合历史记录。
我们的工作对一致性规则的定义是不可知的。
PHANTOM的贡献在于实现了上述的前两个步骤，现在我们来描述它们。

##### A. Intuition  
##### A. 思路 

How can we distinguish between honest blocks (i.e., blocks mined by cooperating nodes) and dishonest ones? 
Recall that the DAG mining protocol instructs a miner to acknowledge in its new block the entire DAG it observes locally, by referencing the “tips” of the DAG. 
Thus, if block B was mined at time t by an honest miner, then any block published before time t−D was received by the miner and is therefore in B’s past set (i.e., referenced by B directly or recursively via its predecessors; see illustration in Figure 1). 
Similarly, if B’s miner is honest then it published B immediately, and so any honest block created after time t+D belongs to B’s future set.
我们如何区分诚实块(即由协作节点挖的块)和不诚实块？  
回想一下，DAG挖矿协议指示矿工通过引用DAG的“末端”，在其新块中确认其本地观察到的整个DAG。
因此，如果块B在时间t由一个诚实的矿工挖出，那么在时间t-D之前发布的任何块都会被该矿工接收，因此会在B的过去集中(即，由B直接引用或通过其祖先递归地引用;参见 图1中的图解)。
同样，如果B的矿工是诚实的，并立即发布了B，所以在时间t + D之后创建的任何诚实的块都属于B的未来集。

As a result, the set of honest blocks in B’s anticone – which we denote anticone h(B)– is typically small, and consists only of blocks created in the interval[t−D,t+D].^2 
In other words, the probability that an honest block B will suffer a large honest anticone is small:
_Pr(|anticone<sub>h</sub>(B)|>k)∈O(e<sup>−C·k</sup>)_ , for some constant C > 0 (this stems from a bound on the Poisson distribution’s tail). 
We rely on this property and set PHANTOM’s parameter k such that the latter probability is smaller than δ , for some predeﬁned δ > 0; 
see discussion in Section 4.  
因此，B的反锥体中的诚实块集- 我们表示为anticone h(B) - 通常很小，并且只包含在时段[t-D，t + D]中创建的块。^2
换句话说，一个诚实的块B会面临一个大的诚实的反锥体的可能性很小：  
对于某个常数C> 0, Pr(|anticone h (B)| > k) ∈ O(e^−C·k)(这源于泊松分布尾部的边界)。  
我们根据这个性质，并且设定PHANTOM的参数k，使得后者的概率小于某个预定义的大于0的δ;
请参阅第4节中的讨论。

Following this intuition, the set of honest blocks (save perhaps a fraction δ thereof) is guaranteed to form a k-cluster.  
遵循这种思路，诚实的块集(可能因此要保存一个分数δ)能保证形成k-集群。

(^2) Note that, in contrast to anticone h(B), an attacker can easily increase the size of anticone(B), for any block B, by creating many blocks that do not reference B and that are kept secret so that B cannot reference them.  
(^2) 注意，与 anticone h(B)相比，对于任何块B,攻击者可以通过创建许多不引用B并且保密的块来很容易地增加 anticone(B)的大小以至于B不能引用它们。

**Definition 1.** Given a DAG G= (C,E), a subset S⊆ C is called a k-cluster, if ∀B ∈S: |anticone(B)∩S|≤k.
**定义1** 给定一个DAG G =(C，E)，如果 ∀B ∈S: |anticone(B)∩S|≤k，则子集S⊆C被称为k-集群。

Note that the attacker can easily create k-clusters as well, e.g., by structuring his blocks in a single chain. 
Fortunately, we can leverage the fact that the group of honest miners possesses a majority of the computational power, and look at the largest k-cluster. 
We argue that the latter represents, in most likelihood, blocks that were mined properly by cooperating nodes. 
Refer to Figure 2 for an illustration of the largest 3 -cluster in a given blockDAG. 
Identifying this set in general DAGs turns out to be computationally infeasible, and so in practice our protocol uses a greedy algorithm which is good enough for our purposes.
We term this selection of a k-cluster a colouring of the DAG, and use the colours blue and red as a convention for blocks inside and outside the chosen cluster, respectively.  
请注意，攻击者也可以很容易地创建k-集群，例如，通过在单个链中构建他的块。
幸运的是，我们可以利用这样一个事实，即诚实的矿工群体拥有大部分计算能力，并且只会考虑最大的k-集群。
我们认为后者很可能代表合作节点正确挖出的区块。
请参考图2，查看给定blockDAG中最大的3-集群的图示。
在一般情况下确定这个集合DAG在计算上是不可行的，所以在实践中我们的协议使用了一个对我们的目标来说足够实用的贪婪算法。
我们称这种K-集群的选择为DAG的着色，并按照惯例将蓝色和红色分别表示所选集群内部和外部块。

##### B. Step #1: recognizing honest blocks
##### B. 步骤 #1：识别诚实的块

Algorithm 1 below selects a k-cluster in a greedy fashion. 
We denote by BLUE<sub>k</sub>(G) the set of blocks that it returns. 
The algorithm operates as follows:  
下面的算法1以贪婪的方式选择k-集群。
我们用BLUE<sub>k</sub>(G)表示它返回的一组块。
该算法操作如下：

> 1) Given a DAG G, the algorithm recursively computes on the past set of each tip in G.^3 
This outputs a k-cluster for each tip.^4 (lines 4-5)
> 2) Then, it makes a greedy choice and picks the largest one among the outputted clusters. (lines 6-7)
> 3) Finally, it tries to extend this set and add to it any block whose anticone is small enough with respect to the set. (lines 8-10)

> 1) 给定一个DAG G，该算法递归计算G 中每个末端的过去集合^3
这将为每个末端输出一个k-集群。^4(第4-5行)
> 2) 然后，它进行一个贪婪的选择，并从输出的集群中选出最大的一个。 (第6-7行)
> 3) 最后，它试图扩展这个集合，并添加任何相对于该集合来说其反锥体足够小的块。 (第8-10行)

(^3) A tip is a leaf-block, that is, a block not referenced by other blocks. See Figure 1.
(^4) Observe that, for any block B, the DAG past(B) is fixed once and for all at B’s creation, and in particular the set BLUE<sub>k</sub>(past(B)) cannot be later modified. 
Thus, in an actual implementation of Algorithm 1, the sets BLUE<sub>k</sub>(B) will have been computed already (by previous calls) and stored, and there will be no need to recompute them.  
(^3) 末端是一个叶块，也就是一个没有被其他块引用的块。 参见图1
(^4) 可见，对于任何块B，DAG的past(B)在B的创建时被永久地固定了，特别是集合BLUE<sub>k</sub>(past(B))不能被过后修改。
因此，在算法1的实际实现中，集合BLUE<sub>k</sub>(B)已经(通过之前的调用)被计算并被存储，并且将不需要重新计算它们。

Intuitively, we first let the DAG inherit the colouring of its highest scoring tip, B<sub>max</sub>, where the score of a block is defined as the number of blue blocks in its past: score(B) :=|BLUE<sub>k</sub>(past(B))|. 
Then, we proceed to colour blocks in anticone(B<sub>max</sub>) in a way that preserves the k-cluster property. 
This inheritance implies that the greedy algorithm operates as a chain-selection rule—B<sub>max</sub> is the chain tip, the highest scoring tip in past(B<sub>max</sub>) is its predecessor in the chain, and so on. 
We denote this chain by Chn(G) = (genesis = Chn<sub>0</sub>(G),Chn<sub>1</sub>(G),...,Chn<sub>h</sub>(G)). 
The reasoning behind this procedure is very similar to that given in Section 1 in relation to the Maximum k-cluster SubDAG problem. 
They only differ in that, instead of searching for the maximal k-cluster, we are hoping to maximize it via the tip with maximal cluster and then adding blocks from its anticone. 
Thus, the reader should think of our algorithm (informally) as approximating the optimal solution to the Maximum k-cluster SubDAG problem.  
直觉上，我们首先让DAG继承其得分最高的末端(B<sub>max</sub>)的着色，其中一个块的得分被定义为过去的蓝色块的数量：score(B) :=|BLUE<sub>k</sub>(past(B))|。
然后，我们继续以保留k-集群属性的方式给anticone（B<sub>max</sub>）着色。
这种继承意味着贪婪算法作为一个链选择规则运行--B<sub>max</sub>是链末端，past(B<sub>max</sub>)中得分最高的是它的上一级，以此类推。
我们用Chn（G）=（genesis = Chn<sub>0</sub>（G），Chn<sub>1</sub>（G），...，Chn<sub>h</sub>（G））表示这条链。
这个过程背后的推理与第1节给出的最大k-集群SubDAG问题非常相似。
它们的区别仅在于，不是搜索最大k-集群，而是希望通过最大集群的末端使其最大化，然后从其反锥体中添加块。
因此，读者应该将我们的算法（非正式地）想象为近似最大k-集群SubDAG问题的最佳解决方案。

![fig 3](https://user-images.githubusercontent.com/22833166/37557316-b6d343a4-2a3d-11e8-8ac2-e66eab0aab45.jpg)

**Fig. 3:** An example of a blockDAG G and the operation of the greedy algorithm to construct its blue set BLUE<sub>k</sub>(G) set, under the parameter k= 3. 
The small circle near each block X represents its score, namely, the number of blue blocks in the DAG past(X). 
The algorithm selects the chain greedily, starting from the highest scoring tip M, then selecting its predecessor K(the highest scoring tip in past(M)),
then H,D(breaking the C,D,E tie arbitrarily), and finally Genesis. 
For methodological reasons, we add to this chain a hypothetical “virtual” block V – a block whose past equals the entire current DAG.
Blocks in the chain(genesis,D,H,K,M,V) are marked with a light-blue shade. 
Using this chain, we construct the DAG’s set of blue blocks,BLUE<sub>k</sub>(G). 
The set is constructed recursively, starting with an empty one, as follows: In step 1 we visit D and add genesis to the blue set (it is the only block in past(D)). 
Next, in step 2, we visit H and add to BLUE<sub>k</sub>(G) blocks that are blue in past(H), namely, C,D,E. 
In step 3 we visit K and add H,I; 
note that block B is in past(K) but was not added to the blue set, since it has 4 blue blocks in its anticone. 
In step 4 we visit M and add K to the blue set; 
again, note that F∈past(M) could not be added to the blue set due its large blue anticone. 
Finally, in step 5, we visit the block virtual(G) =V, and add M and L to BLUE<sub>k</sub>(G), leaving J away due its large blue anticone.  
**图3:** blockDAG G的一个例子，展示贪婪算法在参数k = 3下构造其蓝色集BLUE<sub>k</sub>（G）的操作。
每个块X附近的小圆圈表示其得分，即DAG的past(X)中的蓝色块的数量。
该算法贪婪地选择链，从最高得分尖端M开始，然后选择其上级K（past(M)中的最高得分的末端）
然后H，D（从得分相同的C，D，E中随意选择），最后是创世块。
出于方法论的原因，我们在这条链上添加了一个假设的“虚拟”块V--一个过去与当前DAG相同的块。
链条中的块（创世块，D，H，K，M，V）标有浅蓝色阴影。
使用这个链，我们构建了DAG的蓝色块集合, BLUE<sub>k</sub>（G）。
该集合是递归构造的，从空的开始，如下所示：在第1步中，我们访问D并将创世块添加到蓝色集合（它是past(D)中唯一的块）。
接下来，在步骤2中，我们访问H并添加past(H)中蓝色的块到BLUE<sub>k</sub>(G)块集中，即C，D，E。
在步骤3中，我们访问K并添加H，I;
注意块B已经在past(K)中，但还没有添加到蓝色集合中，因为它的反锥体中有4个蓝色块。
在步骤4中，我们访问M并将K添加到蓝色集合;
再一次请注意，F∈past(M)由于其大蓝色反锥体而无法添加到蓝色集合中。
最后，在步骤5中，我们访问块虚拟（G）= V，并将M和L添加到BLUE<sub>k</sub>（G）中，J由于其大的蓝色反锥体而被抛弃。

We demonstrate the operation of this algorithm in Figure 3. 
Another example appears in Figure 4.
Note that the recursion halts because for any block B∈G:|past(B)|<|G|.
Let us specify the order in which blocks in anticone(B<sub>max</sub>) should be visited, in line 8 of the algorithm. 
We suggest inserting all blocks in anticone(B<sub>max</sub>) into a lexicographical topological priority queue, which we denote topo_queue. 
The priority of a block is represented by the size of its past set;^5 
in case of ties, the block with lowest hash ID is chosen.  
我们在图3中演示该算法的操作。
另一个例子出现在图4中。
请注意，递归会因为任何块B∈G:|past(B)|<|G|而停止。
让我们在算法的第8行中指定应该被访问的anticone(B<sub>max</sub>)中块的顺序。
我们建议将anticone(B<sub>max</sub>)中所有的块插入到一个词典拓扑优先级队列中，我们将其表示为topo_queue。
块的优先级由其过去集的大小表示; ^5
在优先级相同的情况下，选择ID的散列值最小的块。

(^5) This guarantees that a block cannot be popped out while a block in its past is still in the queue, since C∈future(B) =⇒|past(C)|>|past(B)|.
(^5) 由于C∈future(B) =⇒|past(C)|>|past(B)|，所以这确保了块在其过去的块仍然在队列中时不能弹出。

**Remark.** Our choice of ordering via topo_queue is not inherently significant, and many alternative topological orderings can provide similar robustness properties. 
That said, it might be the case that other rules provide faster convergence and confirmation times. 
We revisit this issue in Section 7.  
**备注.** 我们选择通过topo_queue进行排序的本质上并不重要，许多替代拓扑排序可以提供类似的鲁棒性性。
也就是说，其他规则可能会提供更快的收敛和确认时间。
我们在第7节重新讨论这个问题。


To summarize the function that the blue set satisfies, we state the following:
为了总结蓝色集满足的功能，我们陈述如下：

**Proposition 2.** Let _G=G<sub>pub</sub><sup>∞</sup>_ be the eventual DAG containing all blocks in history, and let B be an arbitrary block in G.
**命题2.** 假定 _G = G<sub>pub</sub><sup>∞</sup>_ 是包含历史中所有块的最终DAG，B为G中的任意块。

> - If B was created by an honest miner, the probability that B will not belong to BLUE<sub>k</sub>(G) decreases exponentially with k.
> - If B was created by a malicious miner, and was withheld for a time interval of length T, the probability that B will belong to BLUE<sub>k</sub>(G) decreases exponentially with T.

> - 如果B由诚实的矿工创建，则B不会属于BLUE<sub>k</sub>（G）的概率随k的增大而指数式减小。
> - 如果B由恶意矿工创建，并且在长度为T的时间间隔内被扣留，那么B会属于BLUE<sub>k</sub>(G)的概率将随着T的增大而指数式减小。

The proof of this proposition follows from the proof of Claim 3 in Section 5.
这个命题的证明来自第5节中的断言3的证明。

##### C. Step #2: ordering blocks
##### C. 步骤 #2: 区块排序

Recall that the DAG is ordered partially via its topology. 
We now wish to extend this order to a full order (aka a topological order). 
Our objective is to define a rule that gives precedence to blocks in the blue set, that penalizes blocks outside this set by deferring their location in the order, and that preserves the topological relations of blocks.  
回想一下，DAG是通过其拓扑部分排序的。
我们现在希望将此顺序扩展到全序（又称拓扑顺序）。
我们的目标是定义一个让蓝色集合中的块优先的规则，该规则通过降低顺序来惩罚蓝色集合之外的块，并且保留块的拓扑关系。

We propose the following procedure: Traverse the blue set according to some topological order, and iteratively add blocks to the current last position in ord k; 
when visiting block B, first check if there are blocks in past(B) that haven’t been added yet, add such blocks to the order (again, according to some topological order), and then add B to the order.  
我们提出以下步骤：根据某些拓扑顺序遍历蓝色集合，并迭代地将块添加到ord k中当前的最后位置;
当访问块B时，首先检查past(B)中是否有块尚未添加，将这些块添加到顺序中（再次根据某种拓扑顺序），然​​后将B添加到顺序中。

For example, a possible output of ord<sup>3</sup> on the blockDAG illustrated in Figure 2 is: (A,D,C,G,B,F,I,E,J,H,K).
例如，图2中所示的blockDAG上的ord<sup>3</sup>的可能输出是：（A，D，C，G，B，F，I，E，J，H，K）。

This procedure is formalized in Algorithm 2 below. 
The algorithm begins by initializing an empty priority queue and an empty ordered list. 
Throughout, the list will represent the order in which blocks were popped out from the queue. 
The algorithm favours blocks in the blue set by adding to the queue all of the blue children of the current block. 
When a block is pushed into the queue, all blocks in its past (that weren’t already treated) are pushed as well.  
该过程在下面的算法2中被形式化。
算法首先初始化一个空的优先级队列和一个空的有序列表。
在整个过程中，该列表将表示块从队列中弹出的顺序。
该算法通过向队列中添加当前块的所有蓝色子元素来支持蓝色集中的块。
当一个块被推入队列时，其所有过去的块（尚未处理过的块）也被推入。

In the algorithm,topo_queue is the same lexicographical topological priority queue defined in the preceding subsection. 
In addition, the queue should avoid duplicating elements, and so topo_queue.push(C) should do nothing in case C is already in topo_queue.  
在算法中，topo_queue是在前一小节中定义的同样的词典拓扑优先级队列。
另外，队列应该避免重复元素，所以topo_queue.push(C)在C已经在topo_queue的情况下不应该做任何事情。

The intuition here is simple. 
The algorithm is intended to guarantee that a block B can precede a blue block C only if B is blue, or B is referenced by a blue block. 
In this way, blocks that were withheld by an attacker will not precede blocks that were mined properly and published on time (which are represented roughly by the set of blue blocks).  
这里的思路很简单。
该算法旨在保证块只有B是蓝色，或B是由蓝色块引用时，B才可以在蓝色块C之前。
通过这种方式，攻击者扣留的区块将不会在正确挖出并按时发布的区块（大致由蓝色区块表示）之前。


> **Algorithm 1** Selection of a blue set

**Input:** G – a block DAG, k – the propagation parameter   
**Output:**  BLUE k (G) – the dense-set of G
1. **function** CALC-B L UE(G, k )
2. >   **if** B == genesis **then**
3. >>    **return** {genesis}
4. >   **for** B ∈ tips(G) **do**
5. >>  BLUE<sub>k</sub>(B) ←CALC-BLUE(past (B) , k)
6. >   B<sub>max</sub> ← arg max {|BLUE k (B)| : B ∈ tips(G)} (and break ties arbitrarily)
7. >> BLUE<sub>k</sub>(G) ← BLUE k (B<sub>max</sub>) ∪ {B<sub>max</sub>}
8. >> **for** B ∈ anticone (B max ) **do** in some topological ordering
9. >>> **if** |anticone (B) ∩ BLUE k (G)| ≤ k **then**
10. >>>> add B to BLUE k (G)
11. >> **return** BLUE k (G)

译者注：arg max即“argument of the maximum“的缩写，直译就是”最大值的自变量“，意思是使arg max后面所跟的公式达到最大值的自变量的取值。在上面算法中就是指拥有最多蓝色祖先区块的G的末端区块。

> **Algorithm 2** Ordering of the DAG

<hr>

**Input:** G – a block DAG, k – the propagation parameter   
**Output:**  ord(G) – an ordered list containing all of G’s blocks

1. **function** ORDER(G, k )
2. >   initialize empty queue topo_queue
3. >    initialize empty ordered list L
4. >   BLUE<sub>k</sub>(G) ←CALC-BLUE(G, k)
5. >  topo_queue.push (genesis)
6. >   **while** topo_queue =6 ∅ **do**  
7. >> B ← topo_queue.pop()
8. > L.add(B) (B is added to the end of the list)
9. > **for all** C ∈ childrenB ∩ BLUE<sub>k</sub>(G) **do**
10. >>> **for all** D ∈ past (C) ∩ anticone (b) \ L **do**
11. >>>> topo_queue.push(D)  
12. >>> topo_queue.push(C)
13. > ord(G) ← L
14. > **return** ord(G) 

##### D. Implications to transaction security
##### D. 对交易安全的影响

We now demonstrate how the above procedures of PHANTOM enable safe acceptance of transactions. 
Consider a transaction tx∈B, where B is a block in the blue set of G. 
In order to render tx invalid, a conflicting transaction <ruby>tx<rt>__</rt></ruby> must precede it in the order, and must therefore be embedded in a block C∈anticone(B) that precedes B.^6 
The ordering procedure implies that, for C to precedes B, it must either be a blue block or in the past set of a blue block. 
In both cases,C could not have been withheld for too long, by the second guarantee of Proposition 2.
Thus, the recipient of tx can wait for the blue set around B to become sufficiently robust to reorgs, and then approve tx. 
In Section 5 we prove that robustness is indeed obtained, after some waiting time.  
我们现在演示PHANTOM的上述程序如何确保安全地接受交易。
考虑一个交易tx∈B，其中B是G的蓝色集合中的一个块。
为了使tx无效，冲突交易tx̄ 必须在它的顺序之前，因此必须嵌入在B之前的块 C∈anticone(B) 
排序过程意味着，对于C先于B，它必须是蓝色块或过去的蓝色块。
在这两种情况下，由于命题2的第二个保证，C不能被扣留太久。
因此，tx的接收者可以等待B周围的蓝色集合对重组变得足够强健后，然后批准tx。
在第5节中，我们证明了在一段等待时间之后确实获得了鲁棒性。

#### 3. FORMAL MODEL AND STATEMENT

In this section we describe our formal framework. While we introduce new notation and
terminology, the reader should keep in mind thatwe stick to Bitcoin’s model in almost every
respect—transactions, blocks, Proof-of-work, computationally bounded attacker, P2P propagation
of blocks, probabilistic security guarantees, etc. The “only” difference is that a block references
(possibly) several predecessors rather than a single one. While this has far reaching consequences
on how the ledger is to be interpreted, on the mining side things remain largely the same.

A. Network

We follow the model specified in [8]. The network of nodes (or miners) is denotedN,honest
denotes the set of nodes that abide to the mining protocol (as defined below), andmalicious
denotes the rest of the nodes. Honest nodes form a connected component inN’s topology, and
the communication delay diameter of the honest subnetwork isD: if an honest nodev∈ N
sends a message of sizebMB at timet, it arrives at all honest nodes by timet+Dthe latest.
The attacker is assumed to suffer no delays whatsoever on its outgoing or incoming links.
The real value ofDisa prioriunknown. The PHANTOM protocol assumes thatDis always
smaller than some constantDmax(both depend on the block sizeb). The parameterDmaxis not
hard-coded explicitly in the protocol, rather it influences another parameter,k=k(Dmax), which
is hard-coded and decided once and for all at the inception of the system. Roughly speaking,
k(Dmax) represents an upper bound on the number bound on the number of blocks that the
network creates in one unit of delay and that may not be referenced by one another. Section 4
discusses this parameter in more detail.

B. Mining framework

Proof-of-work.Nodes create blocks of transactions by solving Proof-of-work puzzles. Block
creation follows a Poisson process with parameterλ. For the sake of simplicity, we assume that

(^6) Or,tx ̄can appear inBbeforetx, but this is not an interesting scenario.


λis constant.^7 The computational power of nodev∈ Nis captured by 0 < αv< 1 , which
represents the probability that nodevwill be the creator of the next block in the system (at any
point in time; this is a memoryless process). The attackers’ computational power is less than
50%. Thus,

#### ∑

```
v∈Nαv= 1, and
```
#### ∑

v∈maliciousαv=:α <^0.^5.
Block references.Every block specifies its direct predecessors by referencing their ID in its
header (a block’s ID is obtained by applying a collision resistant hash to its header); the choice
of predecessors will be described in the next subsection. This results in a structure of a direct
acyclic graph (DAG) of blocks (as blocks can only reference blocks created before them), denoted
typicallyG= (C,E). Here,Crepresents blocks andErepresents the hash references. We will
frequently writeB∈Ginstead ofB∈C.

DAG topology.The topology of the blockDAG induces a natural partial ordering over blocks,
as follows: if there is a path in the DAG from block C to blockBwe writeB∈past(C); in
this case,Cwas provably created afterBand thereforeBshould precedeCin the order.^8 A
node does not consider a block as valid until it receives its entire past set. The unique block
genesisis the block created at the inception of the system, and every valid block must have it
in its past set.
Similarly, the future set of a block,future(B), represents blocks that were provably created
after it:B ∈past(C) ⇐⇒ C∈future(B). In contrast to the past set, the future set of
a block keeps growing in time, as more blocks are created and are referencing it. To avoid
ambiguity, we writefuture(B)∩Gorfuture(B,G), and writefuture(B) only when the
context is clear or unimportant.
The setanticone(B) represents all blocks not inB’s future or past (excludingBas well).
These are blocks whose ordering with respect to B is not defined via the partial ordering
that the topology of the DAG induces. Formally, for two distinct blocks B,C ∈ G:C ∈
anticone(B,G) ⇐⇒ (B /∈past(C)∧C /∈past(B)) ⇐⇒ B∈anticone(C,g). Here too
we usually specify the context,anticone(B,G), because the anticone set can grow with time.
In Figure 1 above we illustrates this terminology.

DAG mining protocol.Gvtdenotes the block DAG that nodev∈Nobserves at timet. This DAG
represents the history of all (valid) block-messages received by the node.Goraclet :=∪v∈NGvt
denotes the block DAG of a hypothetical oracle node, andGpubt :=∪v∈honestGvt denotes the
block DAG containing all blocks that are visible to some honest node(s).
Atipof the DAG is a leaf-block, namely, a block with in-degree 0. The instructions to a
miner in the DAG paradigm are simple:

```
1) When creating or receiving a block, transmit it to all of one’s peers inN. Formally, this
implies that∀v,u∈honest:Gvt⊆Gut+D.
```
(^7) In practice,λmust occasionally be readjusted to account for shifting network conditions. PHANTOM can support
a retargeting mechanism similar to Bitcoin’s, e.g., readjust every time thatChn(G) grows by 2016 blocks.
(^8) Note that an edge in the DAG points back in time, from the new block to previously created blocks which it
references.


2) When creating a block, embed in its header a list containing the hash of all tips in the
locally-observed DAG. Formally, this implies that if block B was created at timet, by
honest nodev, thenpast(B) =Gvt.^9
Since these are the only two mining rules in our system, a byzantine behaviour of the attacker
(which controls up toαof the mining power) amounts to an arbitrary deviation from one or
both of these instructions.

C. DAG client protocol

The DAG as described so far possibly embeds conflicting transactions. These are resolved on
the client level. Aclientcan be defined formally as a node inN which has no mining power.
Intuitively, it is any user of the system who is interested in reading and interpreting the current
state of the ledger.
In this work, a transactiontxis an arbitrary message that is embedded in a block. An
underlyingConsistencyrule takes as input a setTof transactions and returnsvalidorinvalid.
Our work is agnostic to the definition and operation of this rule, or to the characterization of the
transaction space. Instead, we focus on the following task: devising a protocol through which
all nodes agree on the order of all transactions in the system. Once such an order is agreed, one
can iterate over all transactions, in the prescribed order, and approve each transaction that is
consistent – according to the underlying rule – with those approved so far.Such an ordering rule
constitutes the client protocol, and is run by each client locally without any need to communicate
additional messages with other clients.
Formally, an ordering ruleordtakes as input a blockDAGGand outputs a linear order over
G’s blocks,ord(G) = (B 0 ,B 1 ,...,B|G|). Transactions in the same block are ordered according
to their appearance in it, and this convention allows us to talk henceforth on the order of blocks
only. With respect to a given ruleord, we writeB≺ord(G) Cif the index ofBprecedes that of
Cinord(G); we abbreviate and writeB≺GCor evenB≺Cwhen the context is understood.
For convenience, we use the same notationB≺GCwhenB∈GbutC /∈G.

D. Convergence of the order

The following definition captures the desired security of the protocol, in terms of the
probability that some order between two blocks will be reversed.

Definition 3.Fix a ruleord. LetB∈G=Gpubt 0. The functionRiskis defined by the probability
that a block that did not precedes Bin timet 1 ≥t 0 will later come to precede it:Risk(B,t 1 ) :=

Pr

#### (

```
∃s > t 1 ,∃C∈Gpubs :B≺Gpubt 1 C∧C≺Gpubs B
```
#### )

#### .

In the definition above, the probability is taken over all random events in the network, including
block creation and propagation, as well as the attacker’s arbitrary (byzantine) behaviour. The
convergence property below guarantees that the order between a block and those succeeding it,

(^9) Technically it is more accurate to writepast(B) =Gvt\{B}, as a block does not belongs to its own past set.


or those not published yet, will not be reversed,w.h.p.This captures the security of the protocol,
as it provides honest nodes with (probabilistic) security guarantees regarding possible reorgs.

Property 1.An ordering ruleordisconvergingif∀t 0 > 0 andB∈Gpubt 0 :tlim
1 →∞

```
Risk(B,t 1 ) =
```
0 , even when a fractionαof the mining power is byzantine.

Remark. Property 1 essentially couples the Safety and Liveness properties required from
consensus protocols. as in Bitcoin and other protocols). Nevertheless, we avoid phrasing our results in these
terms, for the sake of clarity of presentation. The complication arises from the need to analyze
the system from the perspective of every nodeGvt, and not merely from the public ledger’s
hypothetical perspectiveGpubt ; this technicality is not unique to PHANTOM, and should be
regarded in any work that formalizes blockchain based consensus (unless propagation delays
are assumed to be negligible). We leave the task of bridging this gap to a later version.

The security threshold is the minimal hashing power that the attacker must acquire i order to
disrupt the protocol’s operation:

Definition 4.The security threshold of an ordering ruleord is defined as the maximalα
(attacker’s relative computational power) for which Property 1 holds true.

A protocol is scalable if it is safe to increase the block creation rateλwithout compromising
the security, that is, if the security threshold does not deteriorate asλincreases (this can be
phrased also in terms of increasing the block sizebrather thanλ).

E. Main result

Our goal in this paper is to describe formally the ordering procedure of PHANTOM and to
prove that it is scalable in the above sense.

Theorem 5(PHANTOM scales).Given a block creation rateλ > 0 ,δ > 0 , andDmax> 0 , if
Dmaxis equal to or greater than the network’s propagation delay diameterD, then the security
threshold of PHANTOM, parameterized withk(Dmax,δ), is at least 1 / 2 ·(1−δ).

The parameterization of PHANTOM viak(Dmax,δ) is defined in the subsequent section
(see (1)). Theorem 5 encapsulates the main achievement of our work. We prove the theorem
formally in Section 5. Contrast this result to a theorem regarding the Bitcoin protocol, which
appears in several forms in previous work (e.g., [6], [9]):

Theorem 6(Bitcoin does not scale).Asλincreases, the security threshold of the Bitcoin protocol
goes to 0.

Finally, we note that even ifDmax6≥D, the system’s security does not immediately break
apart. Rather, the minimal power needed to attack the system goes from 50% to 0, deteriorating
at a rate that depends on the error gapD−Dmax.


#### 4. SCALABILITY AND NETWORK DELAYS

A. The propagation delay parameterDmax

The scalability of a distributed algorithm is closely tied to the assumptions it makes on the
underlying network, and specifically on its propagation delayD. The real value ofDis both
unknown and sensitive to shifting network conditions. For this reason, Bitcoin operates under
the assumption thatDis much smaller than 10 minutes, and sets the average block interval
time to 10 minutes. While this seems like an overestimation of the network’s propagation delay
under normal conditions (at least in 2018’s Internet terms), some safety margin must be taken,
to account for peculiar network conditions as well. Similarly, in PHANTOM we assume that
the unknownDis upper bounded by someDmaxwhich is known to the protocol. The protocol
does not explicitly encodeDmax, rather, it is parameterized withkwhich depends on it, as will
be described in the next subsection.
The use of ana prioriknown boundDmaxdistinguishes PHANTOM’s security model from
that of SPECTRE [8]. While the security of both protocols depends on the assumption that the
network’s propagation delayDis upper bounded by some constant, in SPECTRE the value of
such a constant need not be known or assumed by the protocol, whereas PHANTOM makes
explicit use of this parameter (viak) when ordering the DAG’s blocks. The fact that the order
between any two blocks becomes robust in PHANTOM, but not in SPECTRE, should be ascribed
to this added assumption; see further discussion in Section 7.

B. The anticone size parameterk

The parameterkis decided from the outset and hard-coded in the protocol. It is defined as
follows:

```
k(Dmax,δ) := (1)
```
```
min
```
#### 

#### 

#### 

```
kˆ∈N:
```
#### (

```
1 −e−^2 ·Dmax·λ
```
#### )− 1

#### ·

#### 

#### 

#### ∑∞

```
j=ˆk+
```
```
e−^2 ·Dmax·λ·
```
```
(2·Dmax·λ) j
j!
```
#### 

```
< δ
```
#### 

#### 

#### 

The motivation here is to devise a bound over the number of blocks created in parallel. Since
the block creation rate follows a Poisson process, for an arbitrary blockBcreated at timet, at
mostk(Dmax,δ) additional blocks were created in the time interval[t−Dmax,t+Dmax], with
probability of at least 1 −δ.^10
Observe that blocks created in the intervals[0,t−Dmax) and(t+Dmax,∞), by honest nodes,
belong toB’s past and future sets, respectively. Consequently, in principle,|anticone(B)|≤k
with probability of 1 −δat least. However, an attacker can artificially increaseB’s anticone by
creating blocks that do not reference it and by withholding his blocks so that Bcannot reference
them.

(^10) In more detail: The second multiplicand in the definition ofkbounds the probability that more thankblocks
were created in parallel toBin the time interval[t−Dmax,t+Dmax]. This term is divided by 1 −e−^2 ·Dmax·λ,
which is the probability that at least one block was created during this time, namely,B. Thus, conditioned on the
appearance ofB, at mostkblocks were created during this time interval, with probability of 1 −δat least.


C. Trade-offs

Theorem 5, and the parameterization of PHANTOM in (1), tie betweenk,Dmax,λ, andδ.
Striving for a better performance by modifying one parameter (e.g., increasingλto obtain larger
throughput and more frequent blocks) must be understood and considered against the effect on
all other parameters.

Increased block creation rate.Although the security threshold does not deteriorate asλis
increased,λcannot be increased indefinitely, or otherwise the network becomes congested. The
value ofλ should be set such that nodes that are expected to participate in the system can
support such a throughput. For instance, if nodes are required to maintain a bandwidth of at
least 1MBper second, and blocks are of sizeb= 1MB, then the block creation rate should
be set toλ= 1blocks per second (this is merely a back-of-the-envelope calculation, and in
practice other messages consume the bandwidth as well).

Higher security threshold.Theorem 5 states the security threshold in terms ofδ. Following (1)
we notice that tightening the security threshold – by choosing a lowerδ– requires increasing
k. A largekleads to slow confirmation times, as will be discussed shortly.^11

Larger safety margin.Similarly, ifDmaxis to be increased, one needs to increasekas well
in order to maintain the same security level (represented byδ).
As discussed in Subsection 4.1, it is better to overestimateDand choose a largeDmaxin order
remain on the safe side.^12 Recall that the security of Bitcoin’s chain depends on the assumption
thatD·λ 1 , namely, that w.h.p. at leastDseconds pass between consecutive blocks, so that
forks are rare. Thus, Bitcoin’s large safety margin overDsuppresses its throughput severely as
it requires selecting a very low block rateλ= 1/ 600 (one block per 10 minutes). This is not
the case with PHANTOM’s DAG, as the security of the DAG ordering does not rely on the
assumptionD·λ 1. Therefore, even if we overestimateD, we can still allow for very high
block creation rates while maintaining the same level of security. Consequently, PHANTOM
supports a very large throughput, and does not suffer from a security-scalability tradeoff.
That said, in PHANTOM there is still a tradeoff between a large safety margin and fast
convergence of the protocol. A gross overestimation ofDmax– resulting an increase ink–
would significantly slow down the waiting time for transaction settlement. Thus,Dmaxshould
be set to a reasonable level. In Section 7 we discuss how this tradeoff can be restricted to visible
conflicts only, and how applications such asPaymentscan enjoy very fast confirmation times
nonetheless.

5. PROOF
We now turn to prove that the order converges, and that an attacker (with less than 50%) is
unable to cause reorgs. We restate the theorem from Section 3:

(^11) The advanced reader should notice that although increasingλhas a similar negative effect onk, it has at the
same time a positive effect on confirmation times, and so a certainλwill be optimal as far as confirmation times are
concerned.
(^12) Several blockchain based projects do not do so, and consequently compromise the security threshold of their
system.


Fig. 4: An example of a DAG with an Hourglass blockK. Here, the delay parameter isk= 3. As
in Figure 3, the small circle near each block represents its score. The colouring of the DAG was done
according to Algorithm 1. The greedily selected chain of blocks of highest score is marked with light-blue
filling (note that this chain is not the longest one). BlockKhas the property that all blue blocks are
either inK’s past or in its future (in addition toKitself). It is thus an Hourglass block.

Theorem 5(PHANTOM scales) Given a block creation rateλ > 0 ,δ > 0 , andDmax> 0 , if
Dmaxis equal to or greater than the network’s propagation delay diameterD, then the security
threshold of PHANTOM, parameterized withk(Dmax,δ), is at least 1 / 2 ·(1−δ).
Our proof relies on the following lemma, which states that if some blockBhas the property
that its anticone contains no blue blocks, then all blue blocks in its past precede all blocks
outside its past. We call this the Hourglass property:

Lemma 7. If for someB̂∈G,BLUE<sub>k</sub>(G)∩anticone

#### (

#### B̂

#### )

```
=∅, then∀B ∈past
```
#### (

#### B̂

#### )

#### ∩

BLUE<sub>k</sub>(G) and∀C /∈past

#### (

#### B̂

#### )

```
,B≺GC. We then writeB̂∈Hourglass(G).
```
In Figure 4 we provide an example of an Hourglass block. The proof of this lemma is
straightforward from the operation of Algorithm 2:

Proof of Lemma 7.First note that if BLUE<sub>k</sub>(G)∩anticone

#### (

#### B̂

#### )

```
=∅thenB̂∈BLUE<sub>k</sub>(G).
```
Indeed, Algorithm 1 defines a chain,Chn(G) (see Subsection 2.2). This chain necessarily

intersects some block inanticone

#### (

#### B̂

#### )

#### ∪

#### {

#### B̂

#### }

. And the intersection block must become blue

itself, by lines (6-7). Thus,BLUE<sub>k</sub>(G)∩anticone

#### (

#### B̂

#### )

```
=∅implies that B̂∈Chn(G) and in
```
particularB̂∈BLUE<sub>k</sub>(G).
Now, Algorithm 2 pushes a block into the queue only after it has pushed already all blocks
in its past (lines 11 and 12). Therefore,topoqueuepops out blocks according to a topological

order. Now, there are no blue blocks inanticone

#### (

#### B̂

#### )

```
, hence ifCis a blue block then it must
```

belong tofuture

#### (

#### B̂

#### )

#### ∪

#### {

#### B̂

#### }

```
, in which case it must have been pushed to the queue afterB̂
```
was popped (unlessC=B̂). Thus, ifCis blue, any block in past

#### (

#### B̂

#### )

```
was popped out before
```
C, and added to the ordered listLbefore it (line 8). In particular,B≺GC. On the other hand,
ifCis not blue, it was necessarily pushed into the queue only by belonging to the past set of
some blue block B′(lines 9-11).B′was popped out afterB̂(unlessB=B̂), from the same
argument as in the previous case. Thus,Cwas pushed into the queue afterB̂was popped out,
which in turn must have occurred afterBwas popped out. In particular, in this case as well,
B≺GC.

Lemma 8.if B̂is an Hourglass block in a DAGG, thenGinherits the order fromB̂:ord(G)∩

past

#### (

#### B̂

#### )

```
=ord(past
```
#### (

#### B̂

#### )

#### ).

Proof.In the proof of the previous lemma we have shown that B̂∈Chn(G). By the recursive
operation of Algorithm 1 (lines 6-7), this implies thatGinherits the colouring ofB̂on its past:
BLUE<sub>k</sub>(G)∩past

#### (

#### B̂

#### )

```
=BLUE<sub>k</sub>
```
#### (

```
past
```
#### (

#### B̂

#### ))

#### .

```
Now, no block inanticone
```
#### (

#### B̂

#### )

```
is pushed intotopo queuebeforeB̂is popped out, because
```
blocks get pushed in only if they are blue (and no such block exists inanticone

#### (

#### B̂

#### )

```
) or if
```
a blue block in their future was pushed in—and this too only happens afterB̂is popped out.

Since the queue respects the topology, this means that all blocks in past

#### (

#### B̂

#### )

```
(which must be
```
visited beforeB̂) are popped out before any block outsidepast

#### (

#### B̂

#### )

#### .

```
The fact that blocks inanticone
```
#### (

#### B̂

#### )

```
are not pushed into the queue before all ofpast
```
#### (

#### B̂

#### )

```
is
```
popped out, implies further that the order in which Algorithm 2 pushes blocks in past

#### (

#### B̂

#### )

```
into
```
and out oftopoqueuedepends only onpast

#### (

#### B̂

#### )

```
, and not on the DAGG. Hence,ord(G)∩
```
past

#### (

#### B̂

#### )

```
=ord(past
```
#### (

#### B̂

#### )

#### ).

The proof of Theorem 5 uses the occurrence of Hourglass blocks to secure all blocks in their
past set.

Proof of Theorem 5. Let δ > 0 and assume that α < 1 / 2 · (1−δ). We need to
prove that ∀t 0 > 0 and B ∈ Gpubt 0 : lim
t 1 →∞

```
Risk(B,t 1 ) = 0, where Risk(B,t 1 ) :=
```
Pr

#### (

```
∃s > t 1 ,∃C∈Gpubs :B≺Gpubt 1 C∧C≺Gpubs B
```
#### )

, and assuming a fraction of at most 1 / 2 −δ
can deviate arbitrarily from the mining protocol.
Fixt 0 andB, and letτ(t 0 ) be the first time aftert 0 where an honest blockB̂was published
such that B̂is an Hourglass block and will forever remain so:

```
τ(t 0 ) := min
```
#### {

```
u≥t 0 :∃B̂∈Gpubu such that B̂∈∩r≥sHourglass
```
#### (

```
Gpubr
```
#### )}

#### (2)


τ(t 0 ) is a random variable. By Lemma 9 it has finite expectation. In particular,

tlim
1 →∞

```
Pr (τ(t 0 )> t 1 ) = 0(e.g., by Markov’s Inequality).
Assume that such aτ(t 0 ) arrives, i.e., that a blockB̂is created and remains permanently
```
an Hourglass block. Then, by Lemma 8, the order between all blocks in past

#### (

#### B̂

#### )

```
never
```
changes, and in particular the order betweenBand any other block Cnever changes. Thus,=
Risk(B,τ(t 0 )) = 0. Therefore, lim
t 1 →∞

```
Pr (τ(t 0 )> t 1 ) = 0implies lim
t 1 →∞
```
```
Risk(B,t 1 ) = 0.
```
Lemma 9.Lett 0 ≥ 0. The expected waiting time forτ(t 0 )(defined in (2)) is finite, and moreover
is upper bounded by a constant that does not depend ont 0.

Proof.LetE(t 0 ) denote the event defined by the following conditions:

```
1) Some blockB̂was created at some timeu > t 0 by an honest node (i.e.,B̂∈Gpubu ) and
apart fromB̂no other block was created in the time interval[u−D,u+D].^13
2) For someT 1 , theklatest blocks inBLUE<sub>k</sub>
```
#### (

```
past
```
#### (

#### B̂

#### ))

```
, denotedLASTk
```
#### (

```
past
```
#### (

#### B̂

#### ))

#### ,

```
were created in the time interval[u−T 1 ,u], and dishonest miners created no blocks in this
time interval.
3) The score of B̂’s chain is forever higher than the score of any chain that does not
pass throughB̂:∀s ≥ u,∀C 1 ,C 2 ∈tips
```
#### (

```
Gpubs
```
#### )

: score(C 1 ) ≥ score(C 2 ) =⇒ B̂ ∈
Chn(past(C 1 )).
Claim 2 and 4 together imply the required result.
The following claim states that the first two conditions in the above definition guarantee that
as long asB̂is blue no block in its anticone is blue:

Claim 1.Assume that for some blockB̂ the first two conditions in the definition ofE(t 0 )
hold true. Then, as long asB̂is blue, all blue blocks haveB̂in their past:∀s≥u,∀B ∈
anticone

#### (

#### B̂

#### )

```
:B /∈BLUE<sub>k</sub>
```
#### (

```
Gpubs
```
#### )

```
∨B /̂∈BLUE<sub>k</sub>
```
#### (

```
Gpubs
```
#### )

#### .

Proof of Claim 1.LetB ∈ anticone

#### (

#### B̂

#### )

. if B /̂ ∈ BLUE<sub>k</sub>

#### (

```
Gpubs
```
#### )

```
we’re done. Assume
```
therefore that B̂∈BLUE<sub>k</sub>

#### (

```
Gpubs
```
#### )

. Any block that was published before timeu−Dbelongs

topast

#### (

#### B̂

#### )

. Any block that was created after timeu+D by an honest node belongs to

future

#### (

#### B̂

#### )

. As honest nodes did not create blocks in the interim, blocks inanticone

#### (

#### B̂

#### )

can only belong to the attacker. Now, since the attacker did not create new blocks while

blocks inLASTk

#### (

```
past
```
#### (

#### B̂

#### ))

```
were created, all attacker blocks that were created in the interval
```
[u−T 1 ,u]and that do not belong to past

#### (

#### B̂

#### )

```
(hence belong to anticone
```
#### (

#### B̂

#### )

```
) have all
```
blocks in LASTk

#### (

```
past
```
#### (

#### B̂

#### ))

```
in their anticone; formally: ∀C ∈ anticone
```
#### (

```
B,Ĝ oracleu
```
#### )

#### :

(^13) We cannot assume, however, that no block was published during this interval, because the attacker might decide
to publish during this interval blocks that he created earlier.


LASTk

#### (

```
past
```
#### (

#### B̂

#### ))

```
⊆anticone
```
#### (

```
C,Goracleu
```
#### )

. Thus, if B̂ ∈ BLUE<sub>k</sub>

#### (

```
Gpubs
```
#### )

```
, any block in
```
anticone

#### (

#### B̂

#### )

```
suffers an anticone that is larger thank(it containsLASTk
```
#### (

```
past
```
#### (

#### B̂

#### ))

#### ∪

#### {

#### B̂

#### }

#### )

and is therefore not inBLUE<sub>k</sub>

#### (

```
Gpubs
```
#### )

. In particular,B /∈BLUE<sub>k</sub>

#### (

```
Gpubs
```
#### )

#### .

Claim 2.The waiting time for the eventE(t 0 ) upper boundsτ(t 0 ).

Proof of Claim 2.The previous claim implies that a blockB̂that satisfies the first two conditions
is an Hourglass block in Gpubs. The third condition implies that B̂forever remains in the chain,
and in particular forever remains blue. It is thus an Hourglass block in all DAGsGpubs (s≥u).

Claim 3.If the first two conditions in the definition ofE(t 0 ) hold true then the third one holds
true with a positive probability.

Proof of Claim 3. Part I:We begin by assuming that the attacker did not publish any block in

the time interval[0,u−Dmax); formally, we assume that∀C∈past

#### (

#### B̂

#### )

:C∈honest.
Let us compare the score of the honest chain to that of any chain that excludesB̂. This gap
is captured by the following definition: For a timer > 0 , define

```
X^1 r:= max
B:B/̂∈Chn(B)
```
```
{score(Chn(B))} (3)
```
```
X^2 r:= max
B:B̂∈Chn(B)
```
```
{score(Chn(B))} (4)
```
```
Xr:=Xr^1 −Xr^2. (5)
```
Let us focus first on the evolution of the processXrbetween time 0 and timeu−T 1. We refer to
the leadXu−T 1 that the attacker obtained at the end of this stage as “the premining gap”; see [8].
LetB 1 rbe the argmax ofXr^1 , and letCr^1 be the latest block in Gpubr ∩BLUE<sub>k</sub>(past(Br 1 )),
namely, the latest honest block which is blue in the attacker chain. Recall that for now we are
assuming that all attacker blocks that were premined were kept secret until after timeu−Dmax.
Observe that at mostkblocks that were created by the attacker beforetime

#### (

```
Cr^1
```
#### )

can be in
BLUE<sub>k</sub>(past(Br 1 )) and can contribute to the score of the attacker’s chain. Thus, between
time

#### (

```
Cr^1
```
#### )

and timeu−T 1 – i.e., during the premining phase – the score of the attacker chain
grows only via the contribution of attacker blocks, and therefore at a rate ofα·λat most.^14
Part II:Let us now consider the growth rate of the honest chain’s score. LetBtmaxbe the tip

of the public chainChn

#### (

```
Gpubt
```
#### )

. The score of this chain is by definition the score of this block.

(^14) Notice that our analysis doesnotassume that the attacker creates its blocks in a single chain. We only claim
that the attacker’s highest scoring chain grows at a rate ofα·λat most, because every attacker block can increase
the attacker’s highest scoring chain by 1 at most. Creating a single chain is indeed the optimal attack on the attacker
side.


Now, by the choice ofk(Dmax,δ)(defined in 1), the probability of an arbitrary honest block B
having too large of an honest anticone is small:^15

```
Pr
B∼arbitrary honest block in Gpubt
```
#### (∣∣

```
∣anticoneh
```
#### (

```
B,Gpubt
```
#### )∣∣

```
∣> k(Dmax,δ)
```
#### )

```
< δ. (6)
```
This is because block creation follows a Poisson process, and because honest blocks that were
createdDmaxseconds before (after) Bbelong to its past (future), hence are not in its anticone.
Since any block that is blue in the honest chain contributes to its score, at any time interval, the
score of the public chain grows at a rate of(1−δ)·(1−α)·λat least. And at mostkhonest
blocks created in the premining stage contributed to the score of the attack chain.
Part III:From Part I and Part II we conclude that the random processXr−kis upper bounded
by the premining race analyzed in [8]. Therein it was shown that the probability distribution
overXu−T 1 −k(the process’s state at the end of the premining stage) is dominated by the
stationary probability distributionπof a reflecting random walk over the non-negative integers
with a bias of(1 1 −−δδ)··(1(1−−αα)) towards negative infinity. The stationary distribution exists because

α≤ 1 / 2 ·(1−δ)<(1−α)·(1−δ).^16
In particular, there is a positive probability that at timeu−T 1 the premining gap,Xu−T 1 , was
less thank, so thatXu−T 1 −k < 0.^17 Between timesu−T 1 anduthe honest network contributed

additionalk+ 1to the score of the public chain, namely, blocksLASTk

#### (

```
past
```
#### (

#### Bˆ

#### ))

#### ∪

#### {

#### B̂

#### }

#### .

During the same time the score of any secret chain(s) did not increase at all, per the second
condition in the definition ofE(t 0 ). Thus,Xu=Xu−T 1 −(k+ 1). And by the first assumption,
Xu+Dmax=Xu. All in all, with some positive probability,Xu+Dmax< 0.
Part IV:Let us turn to look at the evolution of(Xr) r≥u+Dmaxat the second stage. Assume
thatXu+Dmax< 0.

```
In Claim 1 we saw that for anyrsuch that B̂∈BLUE<sub>k</sub>
```
#### (

```
Gpubr
```
#### )

```
, all blocks inB̂’s anticone
```
are red in Gpubr. This implies that, as long asB̂is blue in the public DAG, only attacker blocks

contribute to the score of the attacker’s chain:B̂∈BLUE<sub>k</sub>

#### (

```
Gpubr
```
#### )

```
=⇒∀B∈anticone
```
#### (

#### B̂

#### )

#### :

BLUE<sub>k</sub>(past(B))\future

#### (

#### B̂

#### )

```
=∅(indeed, note that all honest blocks created after time
```
u+Dmaxbelong tofuture

#### (

#### B̂

#### )

```
). Consequently, the attacker’s best chain grows at a rate of
```
α·λat most, as this interval ([u+Dmax,∞)) as well, as long asB̂ ∈ BLUE<sub>k</sub>

#### (

```
Gpubr
```
#### )

#### .

We have already seen that at any time interval the honest chain’s score grows at a rate of
(1−δ)·(1−α)·λat least. Thus, starting at timeu+Dmax, the race between the honest chain’s

(^15) Below,anticoneh(B,G) denotes all blocks inanticone(B,G) created by honest nodes.
(^16) Note that we have multiplied(1−α) by(1−δ) to account for the rare events where there was a burst of block
creation events and where consequently some honest blocks did not contribute to the score of the public chain. Note
on the other hand that blocks created by honest nodes too do not contribute to the attacker chain’s score, even if they
are red in Gpubr. This is because we argued that at mostkblocks that were created by the attacker beforetime
(
Cr^1
)
can be blue inBLUE<sub>k</sub>(past(Br 1 )), and this is regardless ofC^1 r’s status within Gpubr (we merely used the fact that
Cr^1 was created by an honest node, we didn’t use the assumption thatC^1 ris blue in the honest chain).
(^17) In fact, this probability is decreasing logarithmically askincreases.


score and the attacker chain’s score can be modeled as a random walk over all integers with a
bias of(1 1 −−δδ)·(1·(1−−αα)) towards negative infinity.
Since the random walk begins at the negative location Xu+Dmax, this supposedly
implies that with a positive probability the walk will never return to the origin:
Pr (∀r≥u+Dmax:Xr<0) > 0. This in turn implies that B̂ will forever remain a blue
block (and a chain block for that matter). However, to complete the analysis correctly, some
careful attention is required:
Part V:First, we must account for the fact that not all honest nodes observe all honest blocks
immediately, i.e., thatGvr might be a proper subset ofGpubr. This might give the attacker an
advantage, as he can create blocks, reveal them immediately to honest nodes, and these blocks
do not compete with honest blocks unseen yet by these nodes. This advantage can be accounted
for by assuming that the race begins only after the attacker was givenDmaxadditional seconds
to create blocks while the honest network sat idle; see [8]. This fact does not change our general
conclusion thatPr (∀r≥u+Dmax:Xr<0)> 0 , e.g., because there is a positive probability
that the attacker did not create any block during theseDmaxadditional seconds.
Secondly, observe that the honest chain grows according to a random process that is not
necessarily Poisson. Fortunately, a result from [9] shows that nevertheless we can treat the block
race as if the honest score grows according to a Poisson process, and that this assumption can
only increase the value ofXr; it is thus a worst case analysis.
Part VI:Finally, we alleviate our assumption that the attacker published no block during
the premining phase[0,u−Dmax). Instead, consider any attacker block C that belongs to

past

#### (

#### B̂

#### )

. IfC∈BLUE<sub>k</sub>(past(BB)) thenCcontributed to the score of the honest chain

which passes throughB̂and maybe to the attacker chain as well, so its existence does not change
the above analysis. Similarly, ifC /∈BLUE<sub>k</sub>(past(BB)), the fact that it was published has
no consequence whatsoever on the score of the honest chain, and so we can ignore it and apply
the same analysis as if it weren’t published.

Claim 4.The waiting time for the eventE(t 0 ) is finite. Moreover, it is upper bounded by a
constant that does not depend ont 0.

Proof of Claim 4.LetB be an arbitrary honest block created intime(() B). The probability
that no other block was created in the interval( [time(B)−Dmax,time(B) +Dmax]is given by

```
1 −e−^2 ·Dmax·λ
```
#### )− 1

#### ·

#### (

#### 1 −

#### ∑∞

```
j=2e
```
```
− 2 ·Dmax·λ·(e−^2 ·Dmax·λ) j
j!
```
#### )

. An arbitrary block B satisfies the

first condition in the definition ofE(t 0 ) with a positive probability.
Given an arbitrary blockBsatisfying the above condition, letT 1 be the creation time of the
earliest block inLASTk(past(B)). The probability that the attacker did not create any block
in the interval[u−T 1 ,u−Dmax]is given by((1−α)·(1−δ)) k. In particular, given the first
condition, the second one is satisfied with a positive probability.
Importantly, for any two blocksB 1 andB 2 created aftert 0 and that satisfy|time(B 1 )−
time(B 2 )|> 4 ·Dmax, the satisfaction of the first condition with respect toB 1 is independent
from its satisfaction with respect to B 2. Consequently, the expected waiting time for the
occurrence of a blockB̂ which satisfies the first two conditions in the definition of E(t 0 )


is finite (see, for instance, Chapter 10.11 in [10]). Moreover, while the precise expected
time E[Hourglass(t 0 )] may theoretically depend on t 0 , the above argument shows that
E[Hourglass(t 0 )]< const+ 4·d, whereconstdoes not depend ont 0.
This completes the proof of Claim 4 and of Lemma 9.

Theorem 5 guarantees that the probability of reorg with respect to a given blockBdiminishes:
Risk(B,t 1 )→ 0. However, it does not guarantee anything about the convergence rate, i.e., the
waiting time for at 1 that satisfiesRisk(B,t 1 )<  for someI> 0.^18 Following the analysis
in the proof of Claim 4, the waiting time for an Hourglass block can be upper bounded by a
constantin the order of magnitude ofO

#### (

```
eC·Dmax·λ
```
#### )

, for someC > 0 ; and after such a block is
created,the analysis implies thatRisk(B,t 1 ) converges to 0 at an exponential rate, due to the
random walk dynamic.
However, the analysis used in the proof is not tight. It relies on rather infrequent events which
guarantee convergence in a straightforward way, namely, on the occurrence ofHourglassblocks.
In practice the network is likely to converge much faster. Moreover, it can be shown that when
there is no active attack, the convergence time is faster by orders-of-magnitude.

#### 6. VARIANTS

In Section 2 we described the PHANTOM’s greedy algorithm to mark blocks as blue or red
(Algorithm 1). In fact, similar algorithms can provide similar guarantees. We describe several
such variants below and explain the intuition behind them. All these variants can be thought of
as greedy approximations to the Maximumk-cluster SubDAG problem described in Section 1.

A. Choosing the maximizing tip

In our original version of the colouring procedure, we chose the tip which has the highest
score (Algorithm 1, line 6). Instead, the algorithm below chooses the tip for which the score of
the (virtual block of the) current DAG would be highest:

B. Adding blocks to the greedily chosen blue set

Another variant over the previous algorithms is the procedure to mark more blocks as blue,
after the best tip and its blue set were selected. Previously, we required that the candidate block
admit an anticone of size at mostkin the current set. Instead, we can require that its anticone
be counted only inside the originally chosen blue set. Thus, yet another variant is to replace
lines 8-9 with
Observe that the resulting blue set may contain blocks which have an anticone larger thank
within the set.
Yet another viable alternative is to further relax the condition (in lines 8-9) by counting the
anticone of the candidateBonly against the resulting chain of blue blocks,Chn(past(B))∪{B},
where B is the selected tip:

(^18) presents the risk that the party confirming the transaction is willing to absorb.


Algorithm 3Selection of a blue set

Input:G– a block DAG,k– the propagation parameter
Output:BLUE<sub>k</sub>(G)– the dense-set of G
1:function CALC-BLUE(G,k)
2: if B==genesis then
3: return{genesis}
4: for B∈tips(G) do
5: BLUE<sub>k</sub>(B)←CALC-BLUE(past(B),k)
6: SB←BLUE<sub>k</sub>(B)∪{B}
7: for C∈anticone(B) in some arbitrary orderdo
8: if|anticone(C)∩SB|≤k then
9: addC toSB
10: returnarg max{|SB|:B∈tips(G)}(and break ties arbitrarily)

```
8:if|anticone(C)∩(BLUE<sub>k</sub>(B)∪{B})|≤k then
9: addC toSB
```
Compare this variant to Satoshi’s longest-chain rule, which can be describe as follows: each
block in a chainChnincreases its weight by 1, and we select the highest scoring chain. In
contrast, in our last variant, each block whose gap from a chainChnis at mostkincreases its
weight by 1, and we select the highest scoring chain.

C. Iterative elimination of blocks

We now introduce another variant based on an iterative method common in combinatorial
optimization. The algorithm works as follows: Given a block DAGG, we iteratively eliminate
from the blue set the block with largest blue anticone, and continue doing so until all blue blocks
admit a blue anticone of sizekat most:
We conjecture that Algorithm 1 can be replaced with each of the greedy algorithms described
in this section. To this end, suffice it to repeat the proof of Theorem 5 with respect to these
variants. We suspect that the same proof technique, namely, the use of Hourglass, would prove
useful.

#### 7. CONFIRMATION TIMES

As discussed in Section 5, the convergence rate ofRisk(B,t) is slow, at least theoretically
and under certain circumstances. Recall that the functionRisk(B,t) measures the probability

```
8:if|anticone(C)∩(Chn(past(B))∪{B})|≤k then
9: addC toSB
```

Algorithm 4Selection of a blue set

Input:G= (V,E)– a block DAG,k– the propagation parameter
Output:BLUE<sub>k</sub>(G)– the dense-set of G
1:function CALC-BLUE(G,k)
2: BLUE<sub>k</sub>(G)←V
3: while∃B∈BLUE<sub>k</sub>(G) with|anticone(B)∩BLUE<sub>k</sub>(G)|> kdo
4: C←argB<sub>max</sub>{|anticone(B)∩BLUE<sub>k</sub>(G)|}(with arbitrary tie-breaking)
5: removeCfromBLUE<sub>k</sub>(G)
6: return BLUE<sub>k</sub>(G)

that a certain block that did not precedes Bat timetwill later come to precede it. Recall further
that throughput this work we used arbitrary topological orderings (over blue blocks). In light if
this, it would be interesting to seek for an ordering rule (over blue blocks) that would converge
faster. We suspect this is not a trivial task, and leave its full investigation to future work.
The primary factor to the fact that PHANTOM cannot guarantee fast confirmation times is
that membership in the blue set takes time to finalize. The waiting time for such finalization can
be further increased if an attacker manages to balance the decision betweenB∈BLUE<sub>k</sub>(G)
andB /∈BLUE<sub>k</sub>(G). Observe however that if a certain transaction tx∈Badmits no conflicts
inanticone(B), thentxcan be accepted even before the decision regardingB is finalized.

A. Combining SPECTRE and PHANTOM

SPECTRE is a DAG based protocol that can support large transaction throughput (similarly
toPHANTOM) and very fast confirmations. SPECTRE doesnotoutput a linear order over
blocks. Rather, every blockBadmits a vote regarding the pairwise ordering of any two blocks
CandD, and the output is the majority vote regarding each pair. In SPECTRE, cycles of the
sort “BprecedesC,CprecedesD,Dprecedes B” may form.
Furthermore, if an active balancing attack is taking place, forsomepairs of blocks(B,C) the
SPECTRE relation might not become robust (in which caseRisk(B,t)→ 0 is not guaranteed).
Instead, a published blockB is guaranteed to robustly precede any block Cthat was published
later thanB, unlessCwas published shortly afterB. We call this propertyWeak Liveness. In
the context of the Payments application, this fact can only harm a user that signed and published
two conflicting payments at approximately the same time.^19
SincePHANTOM does guarantee (strong) Liveness and a liner ordering, it is interesting
to inquire whether we can enjoy the best of both worlds. We provide below a partial answer to
this question.
Consider the following procedure: Given a blockDAGG,
1) mark blocks as blue or red according to PHANTOM’s colouring procedure

(^19) In contrast, usually any user can engage with a smart contract and introduce conflicts inputs. Thus, Weak Liveness
might potentially harm the usability of SPECTRE to the Smart Contracts application.


2) run SPECTRE on the subDAGBLUE<sub>k</sub>(G); this determines the pairwise ordering between
any twoblueblocksBandC
3) for any blue blockBand red block C, ifC∈past(B) then determine thatCprecedes
B, otherwise determine that BprecedesC
4) decide the pairwise ordering of any two red blocks in some arbitrary way that respects the
topology (B∈past(C)⇒BprecedesC)
Intuitively, we run SPECTRE on the set of blue blocks (as determined by PHANTOM), and
complemented the pairwise ordering in a way that both penalizes red blocks and respects the
topology. We argue that this protocol enjoys SPECTRE’s fast confirmation times and at the
same time inherits the (regular) Liveness property of PHANTOM. To see the latter, observe that
a Hourglass block would have a similar effect in SPECTRE—all blocks in its future will vote
according to its vote. In particular, the pairwise relation of all previous blocks becomes as robust
as the Hourglass block.
Note that this incremental improvement over vanilla SPECTRE is only possible because
we allowed the protocol to assume something on the network’s topology, namely, that the
communication delay diameter is upper bounded byDmax.

B. Summary

In summary, it is possible to achieve both fast confirmation times and Liveness by combining
PHANTOM and SPECTRE. It is of yet unclear whether we can further achieve a linear ordering
without compromising the fast confirmation times. Hopefully, we will provide answers to these
questions in future work.

#### REFERENCES

[1] Ittay Eyal, Adem Efe Gencer, Emin Gun Sirer, and Robbert Van Renesse. Bitcoin-ng: A scalable blockchain ̈
protocol. In13th USENIX Symposium on Networked Systems Design and Implementation (NSDI 16), pages
45–59, 2016.
[2] Michael R Garey and David S Johnson.Computers and intractability, volume 29. wh freeman New York, 2002.
[3] Aggelos Kiayias and Giorgos Panagiotakos. On trees, chains and fast transactions in the blockchain. Cryptology
ePrint Archive, Report 2016/545, 2016.
[4] Eleftherios Kokoris Kogias, Philipp Jovanovic, Nicolas Gailly, Ismail Khoffi, Linus Gasser, and Bryan Ford.
Enhancing bitcoin security and performance with strong consistency via collective signing. In25th USENIX
Security Symposium (USENIX Security 16), pages 279–296. USENIX Association, 2016.
[5] Yoad Lewenberg, Yonatan Sompolinsky, and Aviv Zohar. Inclusive block chain protocols. InInternational
Con
