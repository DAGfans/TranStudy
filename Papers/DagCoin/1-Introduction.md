原文: https://bitslog.files.wordpress.com/2015/09/dagcoin-v41.pdf  

DagCoin is a cryptocurrency design that merges the concepts of transactions and blocks and making each user a miner.
Each transaction carries a proof-or-work and references one or more previous transactions.
The resulting authenticated data structure is a Direct Acyclic Graph (DAG) of transactions where each transaction “confirms” one or more previous transactions.
The confirmation security of a transaction is measured in accumulated amount of proof-of-work referencing the transaction.
This structure is better suited for a cryptocurrency without subsidy (such as a side-chain), since the cost of reversal of a transaction can be easily measured, where in merged-mining the reversal cost depends on the good will of the non-merged hashing power.
 
DagCoin 是一个高度去中心化的加密货币的设计，其整合了交易和区块的概念并能让每一个用户成为矿工.
每笔交易都携带一个工作量证明并引用一个或者多个之前的交易.
生成的已认证的数据结构是由交易组成有向无环图（DAG），其中每笔交易 “确认” 一笔或者多笔之前的交易.
交易的确认安全性是以引用该交易的累计工作量证明来衡量的.
这个结构更适合没有补贴的加密货币（例如侧链），因为逆转交易的成本能很简单地测量，即逆转恶意的已联合的挖矿取决于善意的未联合的算力.

(*译注：这里的补贴其实指的是挖矿奖励或者手续费，参见 https://blockstream.com/sidechains.pdf subsidy 的有关章节*)

One of the problems with the DAG approach is how to limit the maximum cut of the generated DAG
or, in other words, how to prevent all new transactions from referencing the same set of parent transactions, and degenerating the DAG into a star graph.
The DAG must not increase in “width”, and it must “look” more like a yarn under microscope.
I will call this structure a DAG-chain.
  
DAG 方案的其中一个问题怎样限制生成的 DAG 中最大分叉的规模或者换句话说怎样阻止所有新交易引用同样的一组父交易，导致把 DAG 降级为一张星图.
DAG 一定不能增加 “宽度”，并且必须“看上去” 更像一个在显微镜下的纱线（yarn）.
我将称这个结构为 DAG - 链.


![dagcoin-1](https://user-images.githubusercontent.com/22833166/35629520-cb1f9c86-06d9-11e8-9914-f8918476265f.jpg)

A DAG-chain can be informally defined as DAG that:    
一个 DAG - 链可以被非正式地定义为 DAG 需要满足：

- After taking all border (non-parent) nodes k times, it becomes a chain
- 在被所有的边缘（非父）节点引用 k 次后，它成为一条链
(译注: 这里的边缘节点就是叶子节点，可以理解为至少要有K个分支）
- The resulting chain length is proportional to the original node count by a factor close to 2k.
- 生成的链的长度会和原始节点的数量保持因子为大约2k的比例
(译注: 最多有2k个分支）
- If the DAG has more than 2k nodes, you can cut it in two separate DAGs, and the same properties hold for each half (each half having a factor k which is close to the original k factor).
- 如果 DAG 有超过两千节点，你可以将其拆分成两个独立的 DAG，并且每一半保持同样的属性（每一半的因子 k 是和原始 k 因子接近的）

To be able to create a DAG-chain the protocol must prevent users from choosing old transactions to extend the DAG.
Merging branches should be incentivized, but not too much such that users merge the same branches over and over.
The problem of spam is of less importance, as no transaction can get a “free ride” in a block.
We show that the election of an adequate data structure allows the DAG-chain to be formed, but it requires us to change how we think about double-spends.
 
要能创建 DAG - 链，协议必须阻止用户选择旧的交易去拓展 DAG.
合并分支应该被激励，但不能太多以至于用户会反复地合并同一个分支.
垃圾交易的问题不是那么重要，因为没有交易能在区块中 “搭顺风车”（ _译注：交易手续费_ ）.
我们发现，选择一个合适的数据结构可以形成 DAG 链，但这需要我们改变我们对于双重花费的看法.



The premises used to design the DagCoin cryptocurrency are the following:  
设计 DagCoin 加密货币的前提有：

PREMISE: _The cryptocurrency network benefits from creating a DAG-chain growing as “thin” (low k) as possible._  
前提： _加密货币网络受益于创建一个尽可能越来越 “瘦”（低 k）的 DAG - 链._

In other words, having the average maximal cut as low as possible.
Referencing many previous
transactions (high out degree) can make the DAG thinner only if the following transactions reference the transaction with high out degree, but are themselves of low out degree.
The DAG requires high out degree some times, but low out degree another times.
 
换句话说，让平均最大分支尽可能的小.
某笔交易若引用了多笔交易 (高出度), 唯一能使DAG变瘦的方案是该交易被很多交易引用，但这些交易本身的出度很低.
DAG 有时需要高出度, 有时又需要低出度.

DagCoin tries to fulfill that premise, using an incentive structure such that:  
DagCoin 尝试使用如下激励来实现这个前提:

- There is a benefit for users to reference as many previous transactions as possible
-  尽可能多地引用之前的交易对用户来说是有利的
- Referencing many previous transactions is incentivized only when there are many previous transactions unreferenced.
- 只有存在许多未被引用的交易时, 引用许多之前的交易才是被激励的
- There is no competition between users to reference a previous transaction.
- 用户之间没有引用之前交易的竞争
