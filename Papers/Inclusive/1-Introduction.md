# 1 Introduction

Bitcoin, a decentralized digital currency system [9], uses at its core a distributed data structure known as the block chain—a log containing all transactions conducted with the currency. 
Several other distributed systems, such as Ethereum, a general distributed applications platform, have extended Bitcoin’s functionality, yet still rely on a similar block chain to synchronize information between nodes.

比特币是一种去中心化的数字货币系统[9]，其核心采用了被称为区块链的分布式数据结构， 其包含了使用该货币进行的所有交易的日志。
其他几个分布式系统，例如通用的分布式应用程序平台以太坊，已经扩展了比特币的功能，但仍然依靠类似的区块链来在节点之间同步信息。

As Bitcoin, Ethereum, and their likes gain wider acceptance, it is expected that pressure to include more data in their blocks will increase as well. 
Due to bandwidth constraints, larger blocks propagate through the network less eﬃciently, and may thus result in suboptimal performance if too many transactions are included. 
This is mainly due to the uncoordinated creation of blocks by diﬀerent nodes which results in conﬂicts. 
The current protocols dictate that whenever conﬂicts occur, only a single block is adopted, and the others are discarded.

随着比特币，以太坊及其他类似的链得到越来越广泛的承认，预计在其区块中包含更多数据的压力也将增加。
由于带宽限制，较大的块在网络中传播的效率较低，如果包含太多交易，则可能导致性能欠佳。
这主要是由于不同节点无法彼此协调创建区块而导致冲突。
当前的协议规定，每当发生冲突时，仅有一个块被采用，而其他块则被丢弃。

This paper explores an alternative mechanism for the formation of the block chain that is better suited for such protocols when block sizes are large, or when blocks are created often. 
Our modiﬁcation allows the inclusion of transactions from conﬂicting blocks. 
We thus create an incentive for nodes to attempt and include diﬀerent transactions, and thereby increase throughput.

本文探讨了形成区块链的另一种机制，当块大小较大或经出块率较高时，该机制更适合此类协议。
我们的改进允许包含冲突块中的交易。
因此，我们激励节点尽可能打包不同的交易，从而提高吞吐量。

## Conﬂicts, and the structure of the block chain 
The block chain in each protocol is replicated at every node and assists nodes in reaching a consensus on the state of all “accounts”. 
Blocks, which make up the chain, contain an identiﬁer (a cryptographic hash) of their predecessor in the chain, as well as a set of transactions that are consistent according to the state of the ledger represented by the chain they extend. 
To avoid creating a monopoly on the approval of transactions, all nodes have the ability to create blocks. 
To create a block, a node (also known as a miner) has to solve a computationally intense proof of work problem (the proof of work computation essentially consists of guessing inputs to a cryptographic hash function which succeeds only probabilistically). 
Once a block is created, it is distributed to the rest of the network. 
Blocks may be created by diﬀerent nodes at roughly the same time, and may thus extend the same parent block. 
Such blocks may include diﬀerent subsets of transactions, some possibly conﬂicting (conﬂicting transactions are those that move the same money to diﬀerent destinations – they cannot be allowed to co-occur). 
The protocol therefore includes a mechanism for choosing which block survives to extend the chain, while the other conﬂicting ones are eﬀectively ignored. 
The mechanism used by Bitcoin is this: given several extensions of the current chain, pick the longest chain as the version to adopt. 
Ethereum on the other hand uses a diﬀerent selection strategy which is a variant of GHOST [12] (readers unfamiliar with the basic Bitcoin protocol are referred to [9]).

## 冲突和区块链的结构
每种协议都要求在每个节点上复制区块链，并有助于节点就所有“帐户”的状态达成共识。
每个区块链中的块都包含其前驱块的标识符（加密散列），以及根据其所延伸的区块链表示的账本状态而保持一致的一组交易。
为了避免在交易接受时造成垄断，所有节点都可以创建区块。
为了创建一个区块，一个节点（也称为矿工）必须解决一个巨大计算量的工作量证明问题（工作量证明计算本质上是猜测一个密码散列函数的输入，只有一定概率才会猜测成功）。
一旦区块被创建出来，其将被分发到网络的其余部分。
区块可以由不同的节点大致同时创建出来，因此可以扩展同一父区块。
这样的区块可能包括交易的不同子集，有些可能会发生冲突（冲突交易是指将同一笔钱转移到不同地址的交易-是不被允许同时发生的）。
因此，该协议包括一种机制，用于选择保留哪个区块以扩展链，从而有效地忽略其他冲突块。
比特币的机制是这样的：从给定当前链的几个扩展链中，选择最长的链作为采用的版本。
另一方面，以太坊使用了不同的选择策略，这是GHOST的一种变体[12]（不熟悉基本比特币协议的读者可以参考[9]）。

The chain selection rule can be exploited by a malicious node to reverse a payment, an attack known as double-spend. 
The attacker can attempt to build a secret chain of blocks which does not contain the transaction and later, if its chain is long enough, replace the main chain, thereby reversing the payment.

恶意节点可以利用链选择规则来逆转支付，这种攻击称为双花。
攻击者可以尝试构建一个不包含交易的秘密区块链，然后，如果其链足够长，则更换主链，从而推翻该笔支付。

Previous work [6, 12] has shown that with increasing block sizes (or equivalently with increasing block creation rates), more stale (oﬀ-chain) blocks are created.
This, in turn, leads to several problems: First, the security of the protocol against malicious attacks suﬀers.
Second, increases in block size do not translate to linear increases in throughput (as the contents of oﬀ-chain blocks are not included in the ledger).
Finally, the situation in which blocks conﬂict puts smaller less connected miners at a disadvantage: They earn less than their respective share of the rewards, and may be slowly pushed out of the system due to competition with larger miners, a fact which endangers the decentralization of Bitcoin.

先前的工作[6，12]表明，随着块大小的增加（或等效地，出块率的增加），会创建更多的废弃（链外）区块。
反过来，这又导致了几个问题：
首先，针对该协议恶意攻击的安全性。
其次，区块大小的增加不会转化为吞吐量的线性增加（因为账本中不包含链外区块的内容）。
最后，区块冲突的情况使规模较小，连通性较差的矿工处于劣势：他们获得的收益少于其各自应得的份额，并且由于与大型矿工的竞争而可能缓慢地被挤出系统，这危害了比特币的去中心化。

The problems mentioned above form barriers to the scalability of block chain protocols.
If block sizes are not increased, competition between transactions that attempt to enter the block chain will raise fees to high levels that discourage use of the protocol.

上述问题构成了区块链协议扩容的障碍。
如果不增加区块大小，则试图进入区块链的交易之间的竞争将使费用增加到非常高的水平，从而阻碍协议的使用。

Indeed, Ethereum ’s adopted chain selection protocol was speciﬁcally designed to provide stronger security guarantees exactly in these high throughput settings [13], but other issues such as the skewed reward distribution at high rates, or the loss of throughput due to excluded blocks have not been improved.
Our suggested modiﬁcation aims to provide an additional improvement, and works well with GHOST, with its variant used by Ethereum, with the standard longest chain protocol, and in fact, with any protocol that selects a “main” chain. <sup>3</sup>
> <sup>3</sup> For the sake of brevity, we do not go into the details of GHOST or of its Ethereum variant, except where speciﬁcally relevant.

确实，以太坊采用的链选择协议是正是专门为在这些高吞吐量设定中提供更强的安全保证而设计的[13]，但其他问题（例如高出块率时的奖励分配不均或由于排除区块导致的吞吐量损失）尚未改进。（译注：GHOST协议只有主链上的区块即链上区块才会得到奖励，虽然因为提高了出块率会有一定吞吐量和确认时间的提升，但是还是存在大量的非主链即链下区块的浪费，其次由于只有主链区块才能得到奖励，对于同样付出了很大工作量的链下区块而言也是不公平的）
我们提议旨在进一步改进，从而能与GHOST（以太坊使用的变体），或者标准最长链协议以及实际上选择任何“主链”的协议配合使用。<sup>3</sup>
> <sup>3</sup> 为简洁起见，除非特别相关，否则我们不讨论GHOST或其以太坊变体的细节。

The Block DAG, and inclusive protocols 
We propose to restructure the block chain into a directed acyclic graph (DAG) structure, that allows transactions from all blocks to be included in the log.
We achieve this using an “inclusive” rule which selects a main chain from within the DAG, and then selectively incorporates contents of oﬀ-chain blocks into the log, provided they do not conﬂict with previously included content.
An important aspect of the Inclusive protocol is that it awards fees of accepted transactions to the creator of the block that contains them—even if the block itself is not part of the main chain.
Such payments are granted only if the transaction was not previously included in the chain, and are decreased for blocks that were published too slowly.

## Block DAG和Inclusive协议
我们建议将区块链重组为有向无环图（DAG）结构，该结构允许将所有块的交易都包含在日志中。
我们使用“包容性”规则来实现此目的，该规则从DAG中选择一条主链，然后选择性地将链下区块的内容合并到日志中，前提是它们不与先前包含的内容冲突。
Inclusive协议的一个重要的特点是，它向包含交易的区块的创建者奖励已接受交易的费用，即使该区块本身不属于主链。
仅当交易先前未包含在链中时该奖励才有效，并且对于发布太慢的区块降低此类奖励。

Analysis of such strategies is far from simple.
We employ several game theoretic tools and consider several solution concepts making diﬀerent assumptions on the nodes (that they are proﬁt maximizers, cooperative, greedy-myopic, or even paranoid and play safety-level strategies).
In all solution concepts one clear trend emerges: nodes play probabilistically to minimize collisions, and do not choose only the highest fee transactions that would ﬁt into their block.
对此类策略的分析一点也不简单。
我们采用了几种博弈论工具，并考察了几种在节点上做出不同假设的解决方案概念（它们是利益最大化者，合作者，贪婪投机者甚至偏执狂者，并测试了多种安全级别的策略）。
在所有解决方案概念中，都出现了一个明确的趋势：节点会以随机选取交易的方式最大程度地减少冲突，而不是仅选择适合其区块的最高费用的交易。

One potential negative aspect of our suggestion is that attackers that try to double-spend may publish the blocks that were generated in failed attempts and still collect fees for these blocks.
We show that this strategy, which lowers the costs of double-spend attacks, can be easily mitigated with slightly longer waiting times for ﬁnal transaction approval, as the costs of an attacker grow signiﬁcantly with the waiting time. <sup>4</sup> 
We additionally consider a new attack scenario (which has not been analyzed in previous work) in which an attacker creates a public fork in the chain in order to delay transaction acceptance by nodes.
> <sup>4</sup> This is guaranteed only if the attacker has less than 50% of the computational power in the network.

我们建议的一个潜在负面影响是，尝试双花的攻击者可能会发布在失败尝试中生成的区块，并且仍会收取这些区块手续费。
我们发现，尽管这种策略可以降低双花攻击的成本，但是可以通过等待稍长的最终交易接受的时间来轻松缓解，因为攻击者的成本随着等待时间的增加而显着增加。<sup>4</sup>
我们还考虑了一种新的攻击场景（在先前的工作中没有进行过分析），在这种场景下，攻击者在链中创建了一个公开的分叉，以延迟节点接受交易的时间。(译注：这就是一种存活性攻击, 不以推翻交易为目的，而是让链无法正常工作为目的)
> <sup>4</sup> 仅当攻击者的网络计算能力不到50％时，才能保证这一点。

Another issue that arises as many conﬂicting blocks are generated by the protocol, is the problem of selﬁsh mining [7], in which miners deviate from Bitcoin’s proposed strategy to increase their gains.
Inclusive protocols remain susceptible to this form of deviation as well, and do not solve this issue.

该协议因产生许多冲突区块而引起的另一个问题是自私挖矿[7]，在这种情况下，矿工违背了比特币提出的增加收益的策略。
Inclusive协议也容易受到这种形式的背离的影响，并且不能解决该问题。

To summarize, our main contributions are:

1. We utilize a directed acyclic structure for the block graph in which blocks reference several predecessors to incorporate contents from all blocks into the log (similar structures have already been proposed in the past, but not to include the contents of oﬀ-chain blocks).

2. We provide a game theoretic model of the competition for fees between the nodes under the new protocol.

3. We analyze the game under several game theoretic solution concepts and assumptions, and show that in each case nodes randomize transaction selection from a wider range of transactions.
This is the key to the improved performance of the protocol.

4. We demonstrate that Inclusive protocols obtain higher throughput, more proportional outcomes that less discriminate smaller, less-connected players, and that they suﬀer very little in their security in comparison to non-inclusive protocols.
We consider both security against double-spend attempts, as well as attackers that are trying to delay transaction acceptance in the network.

总结一下，我们的主要贡献是：

1. 我们使用有向无环结构组成区块图，其中每个区块引用了多个前驱区块，以将所有块的内容合并到日志中（过去已经提出了类似的结构，但不包括链下的内容）。

2. 我们提供了新协议下节点之间费用竞争的博弈模型。

3. 我们根据几种博弈论的解决方案概念和假设对博弈进行了分析，并表明在每种情况下，节点都会从更大范围的交易中随机选择交易。
这是提高协议性能的关键。

4. 我们证明，Inclusive协议可获得更高的吞吐量，与投入更成比例的收益从而让较小且连通性较弱的节点遭受更少的歧视，与非包容性的协议相比，安全性风险极低。
我们既考虑了防止双重花费攻击尝试的安全性，又考虑了试图延迟网络交易接受的攻击者的安全性。

# References
[6]. Decker, C., Wattenhofer, R.: Information propagation in the Bitcoin network. In:
13th IEEE International Conference on Peer-to-Peer Computing (P2P), Trento, Italy (September 2013)

[7]. Eyal, I., Sirer, E.G.: Majority is not enough: Bitcoin mining is vulnerable. In:
Financial Cryptography and Data Security, pp. 436–454. Springer (2014)

[8]. Kreps, D.M., Wilson, R.: Sequential equilibria. Econometrica: Journal of the Econometric Society pp. 863–894 (1982)

[9]. Nakamoto, S.: Bitcoin: A peer-to-peer electronic cash system (2008)

[10]. Nisan, N., Roughgarden, T., Tardos, E., Vazirani, V.V.: Algorithmic game theory, chap. 9. Cambridge University Press (2007)

[11]. Rosenfeld, M.: Analysis of hashrate-based double spending. arXiv preprint arXiv:1402.2009 (2014)

[12]. Sompolinsky, Y., Zohar, A.: Secure high-rate transaction processing in Bitcoin. In:
Financial Cryptography and Data Security. Springer (2015)