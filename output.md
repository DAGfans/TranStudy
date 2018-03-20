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
<img alt="$past(H) = \left \{ Genesis,C,D,E \right \}$" src="svgs/3006c1a685aa547d338063fc0995eda5.png?invert_in_darkmode" align=middle width="216.994305pt" height="24.6576pt"/> – blocks which H references directly or indirectly, and which were provably created before H;
<img alt="$future(H) =\left \{ {J,K,M} \right \}$" src="svgs/a5167e4797ff065c7cce871479332fa4.png?invert_in_darkmode" align=middle width="171.686955pt" height="24.6576pt"/>– blocks which reference H directly or indirectly, and which were provably created after H;
<img alt="$anticone(H) =\left \{ {B,F,I,L} \right \}$" src="svgs/97edac3cb099b7d1b1b3142a11f3b8f7.png?invert_in_darkmode" align=middle width="194.841405pt" height="24.6576pt"/>– the order between these blocks and H is ambiguous.
Deciding the order between H and blocks in <img alt="$anticone(H)$" src="svgs/fc8e4302109469463c4b37e22d78d47e.png?invert_in_darkmode" align=middle width="90.54375pt" height="24.6576pt"/> is the main challenge of a DAG protocol.
<img alt="$tips(G) =\left \{ {J,L,M} \right \}$" src="svgs/98492cf76392e6c66b98e56a3e138c2f.png?invert_in_darkmode" align=middle width="144.050115pt" height="24.6576pt"/>– leaf-blocks, namely, blocks with in-degree 0; 
these will be referenced in the header of the next block  
**图 1:** 块DAG G的一个例子。  
每个块都会引用矿工在创建时知道的所有块。  
DAG术语，以块H为例子，如下所示：  
<img alt="$past(H) = \{Genesis,C,D,E\}$" src="svgs/fe0ae7dacb2970efb55ad2ed909146ec.png?invert_in_darkmode" align=middle width="216.994305pt" height="24.6576pt"/> - H直接或间接引用，并且很可能在H之前创建的块;  
<img alt="$future(H) =\{J,K,M\}$" src="svgs/53d4e036206b3206a9c5f19ec69b4455.png?invert_in_darkmode" align=middle width="171.686955pt" height="24.6576pt"/> - 直接或间接引用H，并且很可能在H之后创建的块;  
<img alt="$anticone(H) = \{B,F,I,L\}$" src="svgs/12860c7f2cc2cb2aa33b4f427fa0660d.png?invert_in_darkmode" align=middle width="194.841405pt" height="24.6576pt"/> - 这些块与H之间的顺序是不明确的。(译注：后称待序集)
决定H和<img alt="$anticone(H)$" src="svgs/fc8e4302109469463c4b37e22d78d47e.png?invert_in_darkmode" align=middle width="90.54375pt" height="24.6576pt"/>中块之间的顺序是一个DAG协议的主要挑战。  
<img alt="$tips(G) = \{J,L,M\}$" src="svgs/3b7a371ca60556364b6956bb99eb6107.png?invert_in_darkmode" align=middle width="144.050115pt" height="24.6576pt"/> - 叶块，即入度为0的块;
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

> **Maximum** <img alt="$k$" src="svgs/63bb9849783d01d91403bc9a5fea12a2.png?invert_in_darkmode" align=middle width="9.075495pt" height="22.83138pt"/>**-cluster SubDAG** (<img alt="$MCS_k$" src="svgs/6ef9310bccab9c81450c9e3d6381f24f.png?invert_in_darkmode" align=middle width="48.01038pt" height="22.46574pt"/>)  
> **Input:** DAG <img alt="$G= (C,E)$" src="svgs/3a9bb7dc69dcfd836106d5ad14ac8c42.png?invert_in_darkmode" align=middle width="80.027145pt" height="24.6576pt"/>   
> **Output:** A subset <img alt="$S^* \subset C$" src="svgs/8ee2e5405d2225309bb9f6f7290232c7.png?invert_in_darkmode" align=middle width="53.426835pt" height="22.63866pt"/> of maximum size, s.t. <img alt="$|anticone(B)\cap S^*| \leq k$" src="svgs/63c2e6d3c9335e3f7cfd038a019cc03e.png?invert_in_darkmode" align=middle width="165.811305pt" height="24.6576pt"/> for all <img alt="$B \in S^*$" src="svgs/2a115b4f45515da07bf9f09ffd3a5833.png?invert_in_darkmode" align=middle width="51.147195pt" height="22.63866pt"/> .

Here, <img alt="$anticone(B)$" src="svgs/00296720b444aad26fdd661b5a267764.png?invert_in_darkmode" align=middle width="88.837155pt" height="24.6576pt"/> is the set of blocks in the DAG which did not reference B(directly or indirectly via their predecessors) and were not referenced by B(directly or indirectly via B’s predecessors). 
The parameter k is related to an assumption that PHANTOM makes regarding the network’s propagation delay; 
this is explained in detail in Section 4, following the formal framework specified in Section 3.  
这里，<img alt="$anticone(B)$" src="svgs/00296720b444aad26fdd661b5a267764.png?invert_in_darkmode" align=middle width="88.837155pt" height="24.6576pt"/>是DAG中没有引用B(直接或通过它们的祖先间接)并且没有被B引用(直接或通过B的祖先间接)的块集合。
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
Setting PHANTOM’s inter-connectivity parameter with  <img alt="$k= 3$" src="svgs/60a3f0d20317ca09ca3013436ede64cf.png?invert_in_darkmode" align=middle width="39.21225pt" height="22.83138pt"/> means that at most 4 blocks are assumed to be created within each unit of delay, so that typical anticone sizes should not exceed 3.
Blocks outside the largest 3-cluster,E,H,K(coloured red), belong to the attacker (w.h.p.). 
For instance, block E has 6 blue blocks in its anticone (B,C,D,F,G,I); 
these blocks didn’t reference E, presumably because E was withheld from their miners. 
Similarly, block K admits 6 blue blocks in its anticone (B,C,G,F,I,J); 
presumably, its malicious miner received already some blocks from(B,C,D,G), but violated the mining protocol by not referencing them.  
**图2：** 给定DAG中最大的 _3-集群_ 的示例：A，B，C，D，F，G，I，J(蓝色)。
很容易验证这些蓝色块中每个蓝色块在其待序集中最多有3个蓝色块，并且(稍微不容易)该集合是具有该属性的最大集合。
设置PHANTOM的互连参数k = 3意味着最多4个块被假定为在每个延迟单元内创建，因此典型的待序集尺寸不应该超过3。
最大的 _3-集群_ 以外的块E，H，K(红色)属于攻击者(很大可能)。
例如，区块E的待序集有6个蓝色块(B，C，D，F，G，I)。
这些区块没有引用E，大概是因为E被矿工扣留。
类似地，区块K接受在其待序集(B，C，G，F，I，J)中有6个蓝色方块;
据推测，其恶意矿工已经收到了来自(B，C，D，G)的一些块，但没有引用它们而违反了挖矿协议。

##### C. Related work  
##### C. 相关工作

Many suggestions to improve Bitcoin’s scalability have been proposed in recent years. 
These proposals fall into two categories, on-chain scaling and off-chain scaling. 
Roughly speaking, the former includes protocols where all valid transactions are those that appear – as in Bitcoin – inside blocks that are organized in some data structure (aka “the ledger”).  
近年来提出了许多改善比特币可扩展性的建议。
这些建议分为两类，即链上扩容和链下扩容。
粗略地说，前者包括的协议要求所有有效交易出现在以某种数据结构(又称账本)组织的区块内部(如比特币)

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
Bitcoin NG [1]，其中帐本由慢速密钥块(不包含交易)和包含交易的快速微块组成。
Bitcoin NG中密钥块的唯一目的是确定有资格在该时期创建微块的矿工，并确认交易的速度很快。  

GHOST is still susceptible to some attacks, one of which was described in [3]. 
The DAG in Inclusive adds throughput but not security to the main chain, hence suffers from the same limitations as the underlying main chain selection rule. 
Key blocks in Bitcoin NG are still generated slowly, thus confirmation times remain high.  
GHOST仍然容易受到一些攻击，其中之一在[3]中有所描述。
Inclusive中的DAG增加了主链的吞吐量但没有增加安全性，因此会受到与底层主链选择规则相同的限制。
Bitcoin NG中密钥的块仍然会缓慢产生，因此确认时间仍然很长。 

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
Thus, if block B was mined at time t by an honest miner, then any block published before time <img alt="$t-D$" src="svgs/e383f98d3f259d0479682f6cec2112b3.png?invert_in_darkmode" align=middle width="40.093515pt" height="22.46574pt"/> was received by the miner and is therefore in B’s past set (i.e., referenced by B directly or recursively via its predecessors; see illustration in Figure 1). 
Similarly, if B’s miner is honest then it published B immediately, and so any honest block created after time <img alt="$t+D$" src="svgs/3c722f1f836f1019eaa24e8bd8d9a737.png?invert_in_darkmode" align=middle width="40.093515pt" height="22.46574pt"/> belongs to B’s future set.
我们如何区分诚实块(即由协作节点挖的块)和不诚实块？  
回想一下，DAG挖矿协议指示矿工通过引用DAG的“末端”，在其新块中确认其本地观察到的整个DAG。
因此，如果块B在时间t由一个诚实的矿工挖出，那么在时间<img alt="$t-D$" src="svgs/e383f98d3f259d0479682f6cec2112b3.png?invert_in_darkmode" align=middle width="40.093515pt" height="22.46574pt"/>之前发布的任何块都会被该矿工接收，因此会在B的过去集中(即，由B直接引用或通过其祖先递归地引用;参见 图1中的图解)。
同样，如果B的矿工是诚实的，并立即发布了B，所以在时间<img alt="$t+D$" src="svgs/3c722f1f836f1019eaa24e8bd8d9a737.png?invert_in_darkmode" align=middle width="40.093515pt" height="22.46574pt"/>之后创建的任何诚实的块都属于B的未来集。

As a result, the set of honest blocks in B’s anticone – which we denote <img alt="$anticone_h(B)$" src="svgs/4b3d689c446dbbd9a218feaf3da77d5b.png?invert_in_darkmode" align=middle width="97.355115pt" height="24.6576pt"/>– is typically small, and consists only of blocks created in the interval <img alt="$\left [t-D,t+D  \right ]$" src="svgs/96c2e3b26a500b6cd413b35d5318a665.png?invert_in_darkmode" align=middle width="96.625485pt" height="24.6576pt"/> .^2 
In other words, the probability that an honest block B will suffer a large honest anticone is small:
<img alt="$Pr(|anticone_h(B)|&gt;k)\in O(e^{-C\cdot k})$" src="svgs/b13c776cd40e22aeb917f2cecd15d8c9.png?invert_in_darkmode" align=middle width="257.001855pt" height="27.91272pt"/> , for some constant <img alt="$C &gt; 0$" src="svgs/451a003b06aa55fa656256b5a6661b3e.png?invert_in_darkmode" align=middle width="43.061535pt" height="22.46574pt"/> (this stems from a bound on the Poisson distribution’s tail). 
We rely on this property and set PHANTOM’s parameter k such that the latter probability is smaller than δ , for some predeﬁned <img alt="$\delta &gt; 0$" src="svgs/6d5e30d74decbd3932affbbe41aff5b4.png?invert_in_darkmode" align=middle width="38.065005pt" height="22.83138pt"/>; 
see discussion in Section 4.  
因此，B的待序集中的诚实块集- 我们表示为anticone h(B) - 通常很小，并且只包含在时段<img alt="$\left [t-D,t+D  \right ]$" src="svgs/96c2e3b26a500b6cd413b35d5318a665.png?invert_in_darkmode" align=middle width="96.625485pt" height="24.6576pt"/>中创建的块。^2
换句话说，一个诚实的块B会面临一个大的诚实的待序集的可能性很小：  
对于某个常数<img alt="$C&gt; 0$" src="svgs/98fad6c473fcf35c6ddbc6bf05425bfa.png?invert_in_darkmode" align=middle width="43.061535pt" height="22.46574pt"/>, <img alt="$Pr(|anticone_h(B)|&gt;k)\in O(e^{-C\cdot k})$" src="svgs/b13c776cd40e22aeb917f2cecd15d8c9.png?invert_in_darkmode" align=middle width="257.001855pt" height="27.91272pt"/>(这源于泊松分布尾部的边界)。  
我们根据这个性质，并且设定PHANTOM的参数k，使得后者的概率小于某个预定义的大于0的δ;
请参阅第4节中的讨论。

Following this intuition, the set of honest blocks (save perhaps a fraction δ thereof) is guaranteed to form a k-cluster.  
遵循这种思路，诚实的块集(可能因此要保存一个分数δ)能保证形成k-集群。

(^2) Note that, in contrast to anticone h(B), an attacker can easily increase the size of anticone(B), for any block B, by creating many blocks that do not reference B and that are kept secret so that B cannot reference them.  
(^2) 注意，与 anticone h(B)相比，对于任何块B,攻击者可以通过创建许多不引用B并且保密的块来很容易地增加 anticone(B)的大小以至于B不能引用它们。

**Definition 1.** Given a DAG <img alt="$G=(C,E)$" src="svgs/1b61eb9617b2f410434a94cb5d93bd12.png?invert_in_darkmode" align=middle width="80.027145pt" height="24.6576pt"/> , a subset <img alt="$S\subseteq C$" src="svgs/8793ab212272046ae5ba6c08d8f1b68d.png?invert_in_darkmode" align=middle width="45.86967pt" height="22.46574pt"/> is called a k-cluster, if <img alt="$\forall B\in S: |anticone(B)\cap S|\leq k$" src="svgs/1530c03cd4577429286d3c2cc190d0bd.png?invert_in_darkmode" align=middle width="225.496755pt" height="24.6576pt"/>.
**定义1** 给定一个DAG <img alt="$G =(C,E)$" src="svgs/ff92d40356baa785c15f0d3f94162f29.png?invert_in_darkmode" align=middle width="80.027145pt" height="24.6576pt"/>，如果 <img alt="$\forall B \in S: |anticone(B)\cap S|\leq k$" src="svgs/8bfad7b2ebbcb6b7d41bb64359df2ca2.png?invert_in_darkmode" align=middle width="225.496755pt" height="24.6576pt"/>，则子集<img alt="$S\subseteq C$" src="svgs/8793ab212272046ae5ba6c08d8f1b68d.png?invert_in_darkmode" align=middle width="45.86967pt" height="22.46574pt"/>被称为k-集群。

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
We denote by <img alt="$BLUE_k(G)$" src="svgs/0976809cff8421f60e54382f9076863a.png?invert_in_darkmode" align=middle width="83.429445pt" height="24.6576pt"/> the set of blocks that it returns. 
The algorithm operates as follows:  
下面的算法1以贪婪的方式选择k-集群。
我们用<img alt="$BLUE_k(G)$" src="svgs/0976809cff8421f60e54382f9076863a.png?invert_in_darkmode" align=middle width="83.429445pt" height="24.6576pt"/>表示它返回的一组块。
该算法操作如下：

> 1) Given a DAG G, the algorithm recursively computes on the past set of each tip in G.^3 
This outputs a k-cluster for each tip.^4 (lines 4-5)
> 2) Then, it makes a greedy choice and picks the largest one among the outputted clusters. (lines 6-7)
> 3) Finally, it tries to extend this set and add to it any block whose anticone is small enough with respect to the set. (lines 8-10)

> 1) 给定一个DAG G，该算法递归计算G 中每个末端的过去集合^3
这将为每个末端输出一个k-集群。^4(第4-5行)
> 2) 然后，它进行一个贪婪的选择，并从输出的集群中选出最大的一个。 (第6-7行)
> 3) 最后，它试图扩展这个集合，并添加任何相对于该集合来说其待序集足够小的块。 (第8-10行)

(^3) A tip is a leaf-block, that is, a block not referenced by other blocks. See Figure 1.
(^4) Observe that, for any block B, the DAG <img alt="$past(B)$" src="svgs/6e10387941db9c7d0696d2a5c8000e3b.png?invert_in_darkmode" align=middle width="56.68014pt" height="24.6576pt"/> is fixed once and for all at B’s creation, and in particular the set <img alt="$BLUE_k(past(B))$" src="svgs/39d911ef32c9b4743618dfc2a9e5c611.png?invert_in_darkmode" align=middle width="127.184805pt" height="24.6576pt"/> cannot be later modified. 
Thus, in an actual implementation of Algorithm 1, the sets <img alt="$BLUE_k(B)$" src="svgs/f51e673e86a6820541b7c7e11d2344e1.png?invert_in_darkmode" align=middle width="83.79822pt" height="24.6576pt"/> will have been computed already (by previous calls) and stored, and there will be no need to recompute them.  
(^3) 末端是一个叶块，也就是一个没有被其他块引用的块。 参见图1
(^4) 可见，对于任何块B，DAG的<img alt="$past(B)$" src="svgs/6e10387941db9c7d0696d2a5c8000e3b.png?invert_in_darkmode" align=middle width="56.68014pt" height="24.6576pt"/>在B的创建时被永久地固定了，特别是集合<img alt="$BLUE_k(past(B))$" src="svgs/39d911ef32c9b4743618dfc2a9e5c611.png?invert_in_darkmode" align=middle width="127.184805pt" height="24.6576pt"/>不能被过后修改。
因此，在算法1的实际实现中，集合<img alt="$BLUE_k(B)$" src="svgs/f51e673e86a6820541b7c7e11d2344e1.png?invert_in_darkmode" align=middle width="83.79822pt" height="24.6576pt"/>已经(通过之前的调用)被计算并被存储，并且将不需要重新计算它们。

Intuitively, we first let the DAG inherit the colouring of its highest scoring tip, <img alt="$B_{max}$" src="svgs/1fbca5f351d43dfc299de8eb6744ae7e.png?invert_in_darkmode" align=middle width="38.71824pt" height="22.46574pt"/>, where the score of a block is defined as the number of blue blocks in its past: <img alt="$score(B) :=|BLUE_k(past(B))|$" src="svgs/2c3c4076bc4d5d89f9be3d68ca275351.png?invert_in_darkmode" align=middle width="227.194605pt" height="24.6576pt"/>. 
Then, we proceed to colour blocks in <img alt="$anticone(B_{max})$" src="svgs/825b2bb6c8435fba86ccea4fdcadcc46.png?invert_in_darkmode" align=middle width="115.08387pt" height="24.6576pt"/> in a way that preserves the k-cluster property. 
This inheritance implies that the greedy algorithm operates as a chain-selection rule—<img alt="$B_{max}$" src="svgs/1fbca5f351d43dfc299de8eb6744ae7e.png?invert_in_darkmode" align=middle width="38.71824pt" height="22.46574pt"/> is the chain tip, the highest scoring tip in <img alt="$past(B_{max})$" src="svgs/9b5d1e3fa8f0d8c04d63c42ae015ba22.png?invert_in_darkmode" align=middle width="82.92702pt" height="24.6576pt"/> is its predecessor in the chain, and so on. 
We denote this chain by <img alt="$Chn(G) = (genesis = Chn_0(G),Chn_1(G),...,Chn_h(G))$" src="svgs/9e807e2851ffef340d9137665cee4412.png?invert_in_darkmode" align=middle width="402.074805pt" height="24.6576pt"/>. 
The reasoning behind this procedure is very similar to that given in Section 1 in relation to the Maximum k-cluster SubDAG problem. 
They only differ in that, instead of searching for the maximal k-cluster, we are hoping to maximize it via the tip with maximal cluster and then adding blocks from its anticone. 
Thus, the reader should think of our algorithm (informally) as approximating the optimal solution to the Maximum k-cluster SubDAG problem.  
直觉上，我们首先让DAG继承其得分最高的末端(<img alt="$B_{max}$" src="svgs/1fbca5f351d43dfc299de8eb6744ae7e.png?invert_in_darkmode" align=middle width="38.71824pt" height="22.46574pt"/>)的着色，其中一个块的得分被定义为过去的蓝色块的数量：<img alt="$score(B) :=|BLUE_k(past(B))|$" src="svgs/2c3c4076bc4d5d89f9be3d68ca275351.png?invert_in_darkmode" align=middle width="227.194605pt" height="24.6576pt"/>。
然后，我们继续以保留k-集群属性的方式给<img alt="$anticone(B_{max})$" src="svgs/825b2bb6c8435fba86ccea4fdcadcc46.png?invert_in_darkmode" align=middle width="115.08387pt" height="24.6576pt"/>着色。
这种继承意味着贪婪算法作为一个链选择规则运行--<img alt="$B_{max}$" src="svgs/1fbca5f351d43dfc299de8eb6744ae7e.png?invert_in_darkmode" align=middle width="38.71824pt" height="22.46574pt"/>是链末端，<img alt="$past(B_{max})$" src="svgs/9b5d1e3fa8f0d8c04d63c42ae015ba22.png?invert_in_darkmode" align=middle width="82.92702pt" height="24.6576pt"/>中得分最高的是它的上一级，以此类推。
我们用<img alt="$Chn(G) = (genesis = Chn_0(G),Chn_1(G),...,Chn_h(G))$" src="svgs/9e807e2851ffef340d9137665cee4412.png?invert_in_darkmode" align=middle width="402.074805pt" height="24.6576pt"/>表示这条链。
这个过程背后的推理与第1节给出的最大k-集群SubDAG问题非常相似。
它们的区别仅在于，不是搜索最大k-集群，而是希望通过最大集群的末端使其最大化，然后从其待序集中添加块。
因此，读者应该将我们的算法（非正式地）想象为近似最大k-集群SubDAG问题的最佳解决方案。

![fig 3](https://user-images.githubusercontent.com/22833166/37557316-b6d343a4-2a3d-11e8-8ac2-e66eab0aab45.jpg)

**Fig. 3:** An example of a blockDAG G and the operation of the greedy algorithm to construct its blue set <img alt="$BLUE_k(B)$" src="svgs/f51e673e86a6820541b7c7e11d2344e1.png?invert_in_darkmode" align=middle width="83.79822pt" height="24.6576pt"/> set, under the parameter k= 3. 
The small circle near each block X represents its score, namely, the number of blue blocks in the DAG <img alt="$past(X)$" src="svgs/b255e226e88aef38287cf9a520dfe86a.png?invert_in_darkmode" align=middle width="58.29549pt" height="24.6576pt"/>. 
The algorithm selects the chain greedily, starting from the highest scoring tip M, then selecting its predecessor K(the highest scoring tip in <img alt="$past(M))$" src="svgs/fd0da164a9d4f2f2ed8c2f5908c2726a.png?invert_in_darkmode" align=middle width="67.519155pt" height="24.6576pt"/>,
then H,D(breaking the C,D,E tie arbitrarily), and finally Genesis. 
For methodological reasons, we add to this chain a hypothetical “virtual” block V – a block whose past equals the entire current DAG.
Blocks in the chain(genesis,D,H,K,M,V) are marked with a light-blue shade. 
Using this chain, we construct the DAG’s set of blue blocks,<img alt="$BLUE_k(G)$" src="svgs/0976809cff8421f60e54382f9076863a.png?invert_in_darkmode" align=middle width="83.429445pt" height="24.6576pt"/>. 
The set is constructed recursively, starting with an empty one, as follows: In step 1 we visit D and add genesis to the blue set (it is the only block in <img alt="$past(D))$" src="svgs/188c6fb4bab780261cadd720538362ac.png?invert_in_darkmode" align=middle width="63.84576pt" height="24.6576pt"/>. 
Next, in step 2, we visit H and add to <img alt="$BLUE_k(G)$" src="svgs/0976809cff8421f60e54382f9076863a.png?invert_in_darkmode" align=middle width="83.429445pt" height="24.6576pt"/> blocks that are blue in <img alt="$past(H)$" src="svgs/8b6481dde4af0a4cbaa4cceb2c115126.png?invert_in_darkmode" align=middle width="58.386735pt" height="24.6576pt"/>, namely, C,D,E. 
In step 3 we visit K and add H,I; 
note that block B is in <img alt="$past(K)$" src="svgs/6608facc573419bbd6899362e2fd178d.png?invert_in_darkmode" align=middle width="58.52385pt" height="24.6576pt"/> but was not added to the blue set, since it has 4 blue blocks in its anticone. 
In step 4 we visit M and add K to the blue set; 
again, note that <img alt="$F\in past(M)$" src="svgs/14a8e07813170e706796be0915a50f12.png?invert_in_darkmode" align=middle width="94.071615pt" height="24.6576pt"/> could not be added to the blue set due its large blue anticone. 
Finally, in step 5, we visit the block <img alt="$virtual(G) =V$" src="svgs/3ea63ce765f748ce4042a5b045969690.png?invert_in_darkmode" align=middle width="112.22772pt" height="24.6576pt"/>, and add M and L to <img alt="$BLUE_k(G)$" src="svgs/0976809cff8421f60e54382f9076863a.png?invert_in_darkmode" align=middle width="83.429445pt" height="24.6576pt"/>, leaving J away due its large blue anticone.  
**图3:** blockDAG G的一个例子，展示贪婪算法在参数k = 3下构造其蓝色集<img alt="$BLUE_k(G)$" src="svgs/0976809cff8421f60e54382f9076863a.png?invert_in_darkmode" align=middle width="83.429445pt" height="24.6576pt"/>的操作。
每个块X附近的小圆圈表示其得分，即DAG的<img alt="$past(X)$" src="svgs/b255e226e88aef38287cf9a520dfe86a.png?invert_in_darkmode" align=middle width="58.29549pt" height="24.6576pt"/>中的蓝色块的数量。
该算法贪婪地选择链，从最高得分尖端M开始，然后选择其上级K（<img alt="$past(M)$" src="svgs/767ade8f3d468ab2472ed6092f5ebbe2.png?invert_in_darkmode" align=middle width="61.12656pt" height="24.6576pt"/>中的最高得分的末端）
然后H，D（从得分相同的C，D，E中随意选择），最后是创世块。
出于方法论的原因，我们在这条链上添加了一个假设的“虚拟”块V--一个过去与当前DAG相同的块。
链条中的块（创世块，D，H，K，M，V）标有浅蓝色阴影。
使用这个链，我们构建了DAG的蓝色块集合, <img alt="$BLUE_k(G)$" src="svgs/0976809cff8421f60e54382f9076863a.png?invert_in_darkmode" align=middle width="83.429445pt" height="24.6576pt"/>。
该集合是递归构造的，从空的开始，如下所示：在第1步中，我们访问D并将创世块添加到蓝色集合（它是<img alt="$past(D)$" src="svgs/794bf3eb66042c0099f69f5ddae94e63.png?invert_in_darkmode" align=middle width="57.453pt" height="24.6576pt"/>中唯一的块）。
接下来，在步骤2中，我们访问H并添加<img alt="$past(H)$" src="svgs/8b6481dde4af0a4cbaa4cceb2c115126.png?invert_in_darkmode" align=middle width="58.386735pt" height="24.6576pt"/>中蓝色的块到<img alt="$BLUE_k(G)$" src="svgs/0976809cff8421f60e54382f9076863a.png?invert_in_darkmode" align=middle width="83.429445pt" height="24.6576pt"/>块集中，即C，D，E。
在步骤3中，我们访问K并添加H，I;
注意块B已经在<img alt="$past(K)$" src="svgs/6608facc573419bbd6899362e2fd178d.png?invert_in_darkmode" align=middle width="58.52385pt" height="24.6576pt"/>中，但还没有添加到蓝色集合中，因为它的待序集中有4个蓝色块。
在步骤4中，我们访问M并将K添加到蓝色集合;
再一次请注意，<img alt="$F\in past(M)$" src="svgs/14a8e07813170e706796be0915a50f12.png?invert_in_darkmode" align=middle width="94.071615pt" height="24.6576pt"/>由于其大蓝色待序集而无法添加到蓝色集合中。
最后，在步骤5中，我们访问<img alt="$virtual(G) =V$" src="svgs/3ea63ce765f748ce4042a5b045969690.png?invert_in_darkmode" align=middle width="112.22772pt" height="24.6576pt"/>，并将M和L添加到<img alt="$BLUE_k(G)$" src="svgs/0976809cff8421f60e54382f9076863a.png?invert_in_darkmode" align=middle width="83.429445pt" height="24.6576pt"/>中，J由于其大的蓝色待序集而被抛弃。

We demonstrate the operation of this algorithm in Figure 3. 
Another example appears in Figure 4.
Note that the recursion halts because for any block <img alt="$B\in G:|past(B)|&lt;|G|$" src="svgs/072c8b9deed356c7ec285a5b2b4b2ebb.png?invert_in_darkmode" align=middle width="169.794405pt" height="24.6576pt"/>.
Let us specify the order in which blocks in <img alt="$anticone(B_{max})$" src="svgs/825b2bb6c8435fba86ccea4fdcadcc46.png?invert_in_darkmode" align=middle width="115.08387pt" height="24.6576pt"/> should be visited, in line 8 of the algorithm. 
We suggest inserting all blocks in <img alt="$anticone(B_{max})$" src="svgs/825b2bb6c8435fba86ccea4fdcadcc46.png?invert_in_darkmode" align=middle width="115.08387pt" height="24.6576pt"/> into a lexicographical topological priority queue, which we denote <img alt="$topo\_queue$" src="svgs/042bd426dbe7c96a3b8fa81f8aef568a.png?invert_in_darkmode" align=middle width="78.1176pt" height="20.22207pt"/>. 
The priority of a block is represented by the size of its past set;^5 
in case of ties, the block with lowest hash ID is chosen.  
我们在图3中演示该算法的操作。
另一个例子出现在图4中。
请注意，递归会因为任何块<img alt="$B\in G:|past(B)|&lt;|G|$" src="svgs/072c8b9deed356c7ec285a5b2b4b2ebb.png?invert_in_darkmode" align=middle width="169.794405pt" height="24.6576pt"/>而停止。
让我们在算法的第8行中指定应该被访问的<img alt="$anticone(B_{max})$" src="svgs/825b2bb6c8435fba86ccea4fdcadcc46.png?invert_in_darkmode" align=middle width="115.08387pt" height="24.6576pt"/>中块的顺序。
我们建议将<img alt="$anticone(B_{max})$" src="svgs/825b2bb6c8435fba86ccea4fdcadcc46.png?invert_in_darkmode" align=middle width="115.08387pt" height="24.6576pt"/>中所有的块插入到一个词典拓扑优先级队列中，我们将其表示为<img alt="$topo\_queue$" src="svgs/042bd426dbe7c96a3b8fa81f8aef568a.png?invert_in_darkmode" align=middle width="78.1176pt" height="20.22207pt"/>。
块的优先级由其过去集的大小表示; ^5
在优先级相同的情况下，选择ID的散列值最小的块。

(^5) This guarantees that a block cannot be popped out while a block in its past is still in the queue, since <img alt="$C\in future(B) \Rightarrow |past(C)|&gt;|past(B)|$" src="svgs/5f90bc70d8af8ab75ce664c9c7afe1ab.png?invert_in_darkmode" align=middle width="287.941005pt" height="24.6576pt"/>.
(^5) 由于<img alt="$C\in future(B) \Rightarrow |past(C)|&gt;|past(B)|$" src="svgs/5f90bc70d8af8ab75ce664c9c7afe1ab.png?invert_in_darkmode" align=middle width="287.941005pt" height="24.6576pt"/>，所以这确保了块在其过去的块仍然在队列中时不能弹出。

**Remark.** Our choice of ordering via <img alt="$topo\_queue$" src="svgs/042bd426dbe7c96a3b8fa81f8aef568a.png?invert_in_darkmode" align=middle width="78.1176pt" height="20.22207pt"/> is not inherently significant, and many alternative topological orderings can provide similar robustness properties. 
That said, it might be the case that other rules provide faster convergence and confirmation times. 
We revisit this issue in Section 7.  
**备注.** 我们选择通过<img alt="$topo\_queue$" src="svgs/042bd426dbe7c96a3b8fa81f8aef568a.png?invert_in_darkmode" align=middle width="78.1176pt" height="20.22207pt"/>进行排序的本质上并不重要，许多替代拓扑排序可以提供类似的鲁棒性性。
也就是说，其他规则可能会提供更快的收敛和确认时间。
我们在第7节重新讨论这个问题。


To summarize the function that the blue set satisfies, we state the following:
为了总结蓝色集满足的功能，我们陈述如下：

**Proposition 2.** Let <img alt="$G=G_{pub}^\infty $" src="svgs/cb34106cbb7dbd6c14d28cb32c0a3ade.png?invert_in_darkmode" align=middle width="68.09649pt" height="22.46574pt"/> be the eventual DAG containing all blocks in history, and let B be an arbitrary block in G.
**命题2.** 假定 <img alt="$G=G_{pub}^\infty $" src="svgs/cb34106cbb7dbd6c14d28cb32c0a3ade.png?invert_in_darkmode" align=middle width="68.09649pt" height="22.46574pt"/> 是包含历史中所有块的最终DAG，B为G中的任意块。

> - If B was created by an honest miner, the probability that B will not belong to <img alt="$BLUE_k(G)$" src="svgs/0976809cff8421f60e54382f9076863a.png?invert_in_darkmode" align=middle width="83.429445pt" height="24.6576pt"/> decreases exponentially with k.
> - If B was created by a malicious miner, and was withheld for a time interval of length T, the probability that B will belong to <img alt="$BLUE_k(G)$" src="svgs/0976809cff8421f60e54382f9076863a.png?invert_in_darkmode" align=middle width="83.429445pt" height="24.6576pt"/> decreases exponentially with T.

> - 如果B由诚实的矿工创建，则B不会属于<img alt="$BLUE_k(G)$" src="svgs/0976809cff8421f60e54382f9076863a.png?invert_in_darkmode" align=middle width="83.429445pt" height="24.6576pt"/>的概率随k的增大而指数式减小。
> - 如果B由恶意矿工创建，并且在长度为T的时间间隔内被扣留，那么B会属于<img alt="$BLUE_k(G)$" src="svgs/0976809cff8421f60e54382f9076863a.png?invert_in_darkmode" align=middle width="83.429445pt" height="24.6576pt"/>的概率将随着T的增大而指数式减小。

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

We propose the following procedure: Traverse the blue set according to some topological order, and iteratively add blocks to the current last position in <img alt="$ord^k$" src="svgs/ef814b0c706b263e77f17a19bf202a31.png?invert_in_darkmode" align=middle width="31.663005pt" height="27.91272pt"/>; 
when visiting block B, first check if there are blocks in past(B) that haven’t been added yet, add such blocks to the order (again, according to some topological order), and then add B to the order.  
我们提出以下步骤：根据某些拓扑顺序遍历蓝色集合，并迭代地将块添加到<img alt="$ord^k$" src="svgs/ef814b0c706b263e77f17a19bf202a31.png?invert_in_darkmode" align=middle width="31.663005pt" height="27.91272pt"/>中当前的最后位置;
当访问块B时，首先检查past(B)中是否有块尚未添加，将这些块添加到顺序中（再次根据某种拓扑顺序），然​​后将B添加到顺序中。

For example, a possible output of <img alt="$ord^3$" src="svgs/f6babb45d959dc5acac1428cf9a79778.png?invert_in_darkmode" align=middle width="30.949545pt" height="26.76201pt"/> on the blockDAG illustrated in Figure 2 is: (A,D,C,G,B,F,I,E,J,H,K).
例如，图2中所示的blockDAG上的 <img alt="$ord^3$" src="svgs/f6babb45d959dc5acac1428cf9a79778.png?invert_in_darkmode" align=middle width="30.949545pt" height="26.76201pt"/>的可能输出是：（A，D，C，G，B，F，I，E，J，H，K）。

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
In addition, the queue should avoid duplicating elements, and so <img alt="$topo\_queue.push(C)$" src="svgs/8008f62008e6eaba96dfe1c5c7f5b574.png?invert_in_darkmode" align=middle width="143.25135pt" height="24.6576pt"/> should do nothing in case C is already in <img alt="$topo\_queue$" src="svgs/042bd426dbe7c96a3b8fa81f8aef568a.png?invert_in_darkmode" align=middle width="78.1176pt" height="20.22207pt"/>.  
在算法中，topo_queue是在前一小节中定义的同样的词典拓扑优先级队列。
另外，队列应该避免重复元素，所以<img alt="$topo\_queue.push(C)$" src="svgs/8008f62008e6eaba96dfe1c5c7f5b574.png?invert_in_darkmode" align=middle width="143.25135pt" height="24.6576pt"/>在C已经在<img alt="$topo\_queue$" src="svgs/042bd426dbe7c96a3b8fa81f8aef568a.png?invert_in_darkmode" align=middle width="78.1176pt" height="20.22207pt"/>的情况下不应该做任何事情。

The intuition here is simple. 
The algorithm is intended to guarantee that a block B can precede a blue block C only if B is blue, or B is referenced by a blue block. 
In this way, blocks that were withheld by an attacker will not precede blocks that were mined properly and published on time (which are represented roughly by the set of blue blocks).  
这里的思路很简单。
该算法旨在保证块只有B是蓝色，或B是由蓝色块引用时，B才可以在蓝色块C之前。
通过这种方式，攻击者扣留的区块将不会在正确挖出并按时发布的区块（大致由蓝色区块表示）之前。


> **Algorithm 1** Selection of a blue set

**Input:** G – a block DAG, k – the propagation parameter   
**Output:**  <img alt="$BLUE_k(G)$" src="svgs/0976809cff8421f60e54382f9076863a.png?invert_in_darkmode" align=middle width="83.429445pt" height="24.6576pt"/> – the dense-set of G
1. **function** CALC-B L UE(G, k )
2. >   **if** <img alt="$B == genesis$" src="svgs/e3955dc31d69824f32685cf2bfb3e055.png?invert_in_darkmode" align=middle width="102.6762pt" height="22.46574pt"/> **then**
3. >>    **return** {<img alt="$genesis$" src="svgs/58f1fef205fb510d28c46cbaf752144f.png?invert_in_darkmode" align=middle width="54.67968pt" height="21.68331pt"/>}
4. >   **for** <img alt="$B \in  tips(G)$" src="svgs/8b3ba3364cbf1016b609a34b646f1190.png?invert_in_darkmode" align=middle width="86.670045pt" height="24.6576pt"/> **do**
5. >>  <img alt="$BLUE_k(B) \leftarrow CALC-BLUE(past (B) , k)$" src="svgs/66d9fb23461ae7c2fee1c6588210847c.png?invert_in_darkmode" align=middle width="315.250155pt" height="24.6576pt"/>
6. >   <img alt="$B_{max} \leftarrow  \arg   max \{|BLUE_k(B)|:B\in tips(G)\}$" src="svgs/0b31b818083cafb6038a2b87ec89b56e.png?invert_in_darkmode" align=middle width="333.210405pt" height="24.6576pt"/> (and break ties arbitrarily)
7. >> <img alt="$BLUE_k(G) \leftarrow  BLUE_k(B_{max}) \cup   \{B_{max} \}$" src="svgs/e0a5a6e062472af39a0609fec13818f9.png?invert_in_darkmode" align=middle width="293.288655pt" height="24.6576pt"/>
8. >> **for** <img alt="$B \in  anticone (B_{max} )$" src="svgs/e1d3299f0f3bb42bf00ac8429df55739.png?invert_in_darkmode" align=middle width="148.468485pt" height="24.6576pt"/> **do** in some topological ordering
9. >>> **if** <img alt="$|anticone (B) \cap  BLUE_k(G)| \leq  k$" src="svgs/8ea8b444c81ca990c7bd407541cbab16.png?invert_in_darkmode" align=middle width="230.656305pt" height="24.6576pt"/> **then**
10. >>>> add B to <img alt="$BLUE_k(G)$" src="svgs/0976809cff8421f60e54382f9076863a.png?invert_in_darkmode" align=middle width="83.429445pt" height="24.6576pt"/>
11. >> **return** <img alt="$BLUE_k(G)$" src="svgs/0976809cff8421f60e54382f9076863a.png?invert_in_darkmode" align=middle width="83.429445pt" height="24.6576pt"/>

译者注：arg max即“argument of the maximum“的缩写，直译就是”最大值的自变量“，意思是使arg max后面所跟的公式达到最大值的自变量的取值。在上面算法中就是指拥有最多蓝色祖先区块的G的末端区块。

> **Algorithm 2** Ordering of the DAG

<hr>

**Input:** G – a block DAG, k – the propagation parameter   
**Output:**  <img alt="$ord(G)$" src="svgs/98ef863e586444a5f3054ce4cd5f999f.png?invert_in_darkmode" align=middle width="50.107035pt" height="24.6576pt"/> – an ordered list containing all of G’s blocks

1. **function** ORDER(G, k )
2. >   initialize empty queue topo_queue
3. >    initialize empty ordered list L
4. >   <img alt="$BLUE_kG) \leftarrow CALC-BLUE(G, k)$" src="svgs/7feb6046dfaf6d08331af94e0fadf480.png?invert_in_darkmode" align=middle width="264.733755pt" height="24.6576pt"/>
5. >  <img alt="$topo\_queue.push (genesis)$" src="svgs/e7d6c61234e7a765b2e30d6144e2b024.png?invert_in_darkmode" align=middle width="185.005755pt" height="24.6576pt"/>
6. >   **while** <img alt="$topo\_queue \neq \phi$" src="svgs/b1c346220e994a1df6a4b58184fb7425.png?invert_in_darkmode" align=middle width="109.829775pt" height="22.83138pt"/> **do**  
7. >> <img alt="$B \leftarrow  topo\_queue.pop()$" src="svgs/b502be55662ce8bf122709e07c0d7416.png?invert_in_darkmode" align=middle width="158.841705pt" height="24.6576pt"/>
8. > <img alt="$L.add(B)$" src="svgs/5be0e4d0b901ae4061f8d61a7902e216.png?invert_in_darkmode" align=middle width="67.6335pt" height="24.6576pt"/> (B is added to the end of the list)
9. > **for all** <img alt="$C \in  childrenB \cap  BLUE_k(G)$" src="svgs/e8af387676497884df7fde9c1fb93d2b.png?invert_in_darkmode" align=middle width="209.429055pt" height="24.6576pt"/> **do**
10. >>> **for all** <img alt="$D \in  past (C) \cap  anticone (b) \setminus L$" src="svgs/07b9f886bd5c69d15ce110ebb4486fcd.png?invert_in_darkmode" align=middle width="218.043705pt" height="24.6576pt"/> **do**
11. >>>> <img alt="$topo\_queue.push(D)$" src="svgs/48797bfbe30bf2aa88e6792ffc386694.png?invert_in_darkmode" align=middle width="144.39282pt" height="24.6576pt"/>
12. >>> <img alt="$topo\_queue.push(C)$" src="svgs/8008f62008e6eaba96dfe1c5c7f5b574.png?invert_in_darkmode" align=middle width="143.25135pt" height="24.6576pt"/>
13. > <img alt="$ord(G) \leftarrow  L$" src="svgs/65453870600c8d7143323339b7f1b202.png?invert_in_darkmode" align=middle width="86.86491pt" height="24.6576pt"/>
14. > **return** <img alt="$ord(G)$" src="svgs/98ef863e586444a5f3054ce4cd5f999f.png?invert_in_darkmode" align=middle width="50.107035pt" height="24.6576pt"/>

##### D. Implications to transaction security
##### D. 对交易安全的影响

We now demonstrate how the above procedures of PHANTOM enable safe acceptance of transactions. 
Consider a transaction <img alt="$tx\in B$" src="svgs/f646e5ce87a606461a7efab656794cf8.png?invert_in_darkmode" align=middle width="48.715755pt" height="22.46574pt"/>, where B is a block in the blue set of G. 
In order to render tx invalid, a conflicting transaction <img alt="$\bar{tx}$" src="svgs/12a150152ba79b7a5bd8f22a33b94897.png?invert_in_darkmode" align=middle width="15.33114pt" height="24.7335pt"/> must precede it in the order, and must therefore be embedded in a block <img alt="$C\in anticone(B)$" src="svgs/f2a673993bb0d7fb38e10f78686a6c69.png?invert_in_darkmode" align=middle width="121.85283pt" height="24.6576pt"/> that precedes B.^6 
The ordering procedure implies that, for C to precedes B, it must either be a blue block or in the past set of a blue block. 
In both cases,C could not have been withheld for too long, by the second guarantee of Proposition 2.
Thus, the recipient of tx can wait for the blue set around B to become sufficiently robust to reorgs, and then approve tx. 
In Section 5 we prove that robustness is indeed obtained, after some waiting time.  
我们现在演示PHANTOM的上述程序如何确保安全地接受交易。
考虑一个交易tx\in B，其中B是G的蓝色集合中的一个块。
为了使tx无效，冲突交易<img alt="$\bar{tx}$" src="svgs/12a150152ba79b7a5bd8f22a33b94897.png?invert_in_darkmode" align=middle width="15.33114pt" height="24.7335pt"/> 必须在它的顺序之前，因此必须嵌入在B之前的块<img alt="$C\in anticone(B)$" src="svgs/f2a673993bb0d7fb38e10f78686a6c69.png?invert_in_darkmode" align=middle width="121.85283pt" height="24.6576pt"/>
排序过程意味着，对于C先于B，它必须是蓝色块或过去的蓝色块。
在这两种情况下，由于命题2的第二个保证，C不能被扣留太久。
因此，tx的接收者可以等待B周围的蓝色集合对重组变得足够强健后，然后批准tx。
在第5节中，我们证明了在一段等待时间之后确实获得了鲁棒性。

