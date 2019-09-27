# 2 From Trees to Directed Acyclic Graphs (DAGs)
# 2 从树到有向无环图（DAG）

We now begin to describe our proposed changes to the protocol.
We start with a structural change to the blocks that will enable further modiﬁcations.
In the current Bitcoin protocol, every block points at a single parent (via the parent’s hash), and due to natural (or malicious) forks in the network, the blocks form a tree.

现在我们开始描述对协议建议的更改。
我们从对区块的结构改动开始，以实现进一步的修改。
在当前的比特币协议中，每个区块都指向单个父区块（通过父节点的散列），并且由于网络中的自然（或恶意）分叉，这些块形成了一棵树。

We propose, instead, the node creating the block would list all childless blocks that it was aware of.
Surely, this added information does not hurt; it is simple to trace each of the references and see which one leads, for example, to the longest chain.
We thus obtain a directed acyclic graph (DAG) in which each block references a subset of previous blocks.
We assume that when block C references B, C’s creator knows all of B’s predecessors (it can request them).
The information that can be extracted from a block’s reference list is suﬃcient to simulate the underlying chain selection rule: we can simulate the longest-chain rule, for example, by recursively selecting in each block a single link—the one leading to the longest chain.

相反，我们建议，创建区块的节点将列出它知道的所有无子区块。
当然，这些增加的信息不会产生副作用；跟踪每个引用并查看哪个引用指向最长的链是很简单的。。
因此，我们得到一个有向无环图(DAG)，其中每个块都引用了先前块的子集。
我们假设，当块C引用B时，C的创建者知道B的所有祖先区块（可以请求他们）。
从区块的引用列表中提取的信息足以模拟基本的链选择规则：例如，我们可以通过在每个块中递归选择出指向最长链一条链来模拟最长链规则。（译注：即父区块中选择难度最大区块， 然后再从该区块的父亲中选择难度最大的，依次类推直到创世区块，选出区块链则为最长链）

The provision of this additional information amounts to a “direct revelation mechanism”: Instead of instructing nodes to select the chain they extend, we simply ask them to report all possible choices, and other nodes can simulate their choice, just as they would have made it (the term direct revelation is borrowed from economics where it is widely used in mechanism design [10]).

提供这些附加信息就构成了“直接启示机制”：我们没有指示节点选择它们扩展的链，而是仅要求它们报告所有可能的选择，而其他节点可以模拟它们的选择（直接启示一词是从经济学中借用的，它在机制设计中被广泛使用[10]）。(译注：这里的意思是你因为拥有完整的信息，则你可以通过算法推导出最长链，而非因为信息缺失，无法推导出最长链，只能指导节点哪条是最长链)

In fact, any chain selection protocol can be simulated in this manner, as the references provide all information needed to determine the choice that the block creator would have made when extending the chain.
The only issue that needs to be handled is tie breaking (as in the case of conﬂicting chains of equal length).
To do so, we ask nodes to list references to other blocks in some order, which is then used to break ties.
Note that nodes are only required to list the childless nodes in the DAG; there is no need to list other nodes, as they are already reachable by simply following the links.
<sup>5</sup>

> <sup>5</sup> DAGs are already required by GHOST (although for diﬀerent reasons), and Ethereum’s blocks currently reference parent blocks as well as “uncles” (blocks that share the same parent as their parent).
Thus, this modiﬁcation is quite natural.

实际上，可以以这种方式模拟任何链选择协议，因为引用提供了确定在扩展链时区块创建者将做出的选择所需的所有信息。
唯一需要处理的问题是解决平局（如冲突等长链的情况）。
为此，我们要求节点以某种顺序列出对其他块的引用，然后将其用于解决平局。
请注意，仅要求节点列出DAG中的无子区块；无需列出其他区块，因为只需通过引用即可访问其他区块。
<SUP>5</SUP>

> <sup> 5 </sup> GHOST已经在使用DAG了（尽管出于不同的原因），以太坊的区块当前引用了父区块以及“叔块”（与父区块共享同一父区块的区块）。
因此，这种修改是很自然的。（译注：按照GHOST原本的设定，区块只会有单一父亲，所以会形成一颗树，但是Ethereum的变种中因为区块会同时引用叔块，则代表有多个父区块，此时已经是DAG了）

Formally, we denote by BDAG the set of all directed acyclic block graphs G = (V, E) with vertices V (blocks) and directed edges E, where each B ∈ V has in addition an order ≺B over all its outgoing edges.
In our setup, an edge goes from a block to its parent, thus childless vertices (“leaves”) are those with no incoming edges.
Graphs in BDAG are required to have a unique maximal vertex, “the genesis block”.
We further denote by sub(B, G) the subgraph that includes all blocks in G reachable from B.


形式上，我们用BDAG表示所有有向无环图G =（V，E）的集合，其顶点为V（块），有向边为E，其中每个B∈V相对其输出边来说都存在≺B的顺序。（译注：即B的顺序比其所有的后代都优先）
在我们的设定中，边从区块指向其父顶点，因此无子顶点（“叶顶点”）是没有传入边的。
BDAG的图中必须具有唯一的最大顶点，即“创世块”。
我们后面用sub（B，G）表示包含从B可达的G中所有块的子图。

An underlying chain selection rule F is used to decide on the main chain in the DAG (e.g., longest-chain or GHOST).
The rule F is a mapping from block DAGs to block chains such that for any G ∈ BDAG, F(G) is a maximal (i.e., non-extendable) chain in G.
The order ≺<sub>B</sub> is assumed to agree with F, in the sense that if A is one of B’s parents and A ∈ F(sub(B, G)), then A is ﬁrst in the order ≺<sub>B</sub> .

基础链选择规则F用于确定DAG中的主链（例如，最长链或GHOST）。
规则F是从区块图到区块链的映射，因此对于任何G∈BDAG，F（G）是G中的最大（即不可扩展）链。
假定≺<sub>B</sub> 也遵守F规则，如果A是B的父区块之一且A∈F（sub（B，G）），则A是≺<sub>B</sub>顺序中最靠前的。


## 2.1 Exploiting the DAG Structure—The Inclusive Protocol
## 2.1 开发DAG结构-Inclusive协议

We deﬁne Inclusive-F, the “Inclusive” version of the chain selection rule F, which incorporates non-conﬂicting oﬀ-chain transactions into a given blocks accepted transaction set.
Intuitively, a block B uses a postorder traversal on the block DAG to form a linear order on all blocks.
If two conﬂicting transactions appear, the one that appeared earlier according to this order is considered to be the one that has taken place (given that all previous transactions it depends on have also occurred).
Thus, we use the order on links that blocks provide to deﬁne an order on blocks, which we then use to order transactions that appear in those blocks, and ﬁnally, we conﬁrm transactions according to this order.


我们定义了包容性F，它是链选择规则F的“包容性”版本，它将非冲突的o链交易合并到给定的块接受交易集中。
直观地，块B使用块DAG上的后序遍历在所有块上形成线性顺序。
如果出现两个冲突的事务，则按照该顺序出现的较早的事务将被视为已发生的事务（假定它所依赖的所有先前事务也已发生）。
因此，我们使用块提供的链接上的顺序来定义块上的顺序，然后使用该顺序对出现在那些块中的交易进行排序，最后，我们根据该顺序确认交易。


To make the Inclusive algorithm formal, we need to provide a method to decide precisely the set of accepted transactions.
Bitcoin transactions are composed of inputs (sources of funds) and outputs (the targets of funds).
Outputs are, in turn, spent by inputs that redirect the funds further.
We deﬁne the consistency of a transaction set, and its maximality as follows:

为了使包容性算法正式化，我们需要提供一种方法来精确确定接受交易的集合。
比特币交易由投入（资金来源）和产出（资金目标）组成。
反过来，产出又被投入所花费，而投入又使资金进一步转移。
我们定义事务集的一致性及其最大值，如下所示：

### Deﬁnition 1.

Given a set of transactions T, a transaction tx is consistent with T if all its inputs are outputs of transactions in T, and no other transaction in T uses them as inputs.
We say that T is consistent, if every transaction tx ∈ T is consistent with T \ {tx}.


### 定义1。

给定一组事务T，如果事务tx的所有输入都是T中事务的输出，并且T中没有其他事务将其用作输入，则事务tx与T一致。
如果每个事务tx∈T与T \ {tx}一致，我们说T是一致的。


### Deﬁnition 2.

We say that a consistent set of transactions T from a block DAG G is maximal, if no other consistent set T ′ of transactions from G contains T.

The algorithm below performs a postorder traversal of the DAG sub(B, G).
Along its run it conﬁrms any transaction that is consistent with those accepted thus far.
The traversal backtracks if it visits the same block twice.
<sup>6</sup>

> <sup>6</sup> It is important to note that the algorithm below describes a full traversal.
More eﬃcient implementations are possible if a previously traversed DAG is merely being updated (with methods similar to the unspent transaction set used in Bitcoin).


### 定义2。

我们说，如果没有其他来自G的交易的一致集合T'包含T，则来自块DAG G的交易T的一致集合是最大的。

以下算法执行DAG子（B，G）的后序遍历。
在运行过程中，它确认与迄今接受的交易一致的任何交易。
如果遍历同一块两次，则遍历回溯。
<SUP>6</SUP>

> <sup> 6 </sup>请务必注意，以下算法描述了完整遍历。
如果仅更新先前遍历的DAG（使用类似于比特币中未使用的交易集的方法），则可能有更有效的实现方式

If B is the genesis block, which has no predecessors, m = 0.


如果B是没有任何前身的创世块，则m = 0。


The algorithm is to be called with arguments Inclusive-F(G, B, ∅), initially setting visited(·) as False for all blocks.
Its output is the set of transactions it approves.


该算法将使用参数Inclusive-F（G，B，∅）进行调用，最初将所有块的visited（·）设置为False。
其输出是它批准的交易集。


### Algorithm 1.
Inclusive-F(G, B, T)

Input: a DAG G, a block B with pointers to predecessors (B 1 , ..., B m ) (ordered according to ≺<sub>B</sub> ), <sup>7</sup> and a set of previously conﬁrmed transactions T.


> <sup>7</sup> If B is the genesis block, which has no predecessors, m = 0.


1. >  IF visited(B) RETURN T
2. >  SET visited(B):=True
3. >  FOR i = 1 TO m:
4. >>  T = Inclusive-F(G, B<sub>i</sub> , T)
5. >  FOR EACH tx ∈ B
6. >>  IF (tx is consistent with T) THEN T = T ∪ {tx}
7. >  RETURN T

We say that B is a valid block if at the end of the run on sub(B, G) we have B ⊆ T.
<sup>8</sup>  The algorithm’s run extends ≺<sub>B</sub> to a linear order on sub(B, G), deﬁned by: A ≺<sub>B</sub> A′ if Inclusive-F(G, B, ∅) visited A before it visited A′ .
The following proposition states that the algorithm provides consistent and maximal transaction sets:

> <sup>8</sup> The Inclusive algorithm can also handle blocks that have some of their transactions rejected.


我们说如果在sub（B，G）上运行的末尾有B⊆T，则B是有效的块。
<sup> 8 </sup>算法的运行将extends <sub> B </sub>扩展为 sub（B，G）上的线性顺序，定义为：A≺<sub> B </sub> A'如果Inclusive-F（G，B，∅）在访问A'之前访问过A。
以下命题指出该算法提供了一致且最大的交易集：

> <sup> 8 </sup>包含算法也可以处理某些交易被拒绝的块。


### Proposition 1.

Let T be the set returned by Inclusive-F(G, B, ∅).
Then T is both consistent and maximal in sub(B, G).


### 命题1。

令T为Inclusive-F（G，B，∅）返回的集合。
那么在sub（B，G）中，T既是一致的又是最大值。


The proof is immediate from the algorithm.


证明是直接从算法中得出的。


An important property of this protocol is that once a transaction has been approved by some main chain block B of G, it will remain in the approved set of any extending block as long as B remains in G’s main chain.
This is because transactions conﬁrmed by main chain blocks are ﬁrst to be included in the accepted transaction sets of future main chain blocks.
Since both in longest-chain and GHOST blocks that are buried deep in the main chain become increasingly less likely to be replaced, the same security guarantees hold for transactions included in their Inclusive versions.


该协议的一个重要特性是，一旦交易被G的某个主链B批准，只要B保留在G的主链中，该交易就会保留在任何扩展块的批准集中。
这是因为首先将由主链块确认的交易包括在将来的主链块的公认交易集中。
由于埋在主链深处的最长链块和GHOST块变得越来越不可能被替换，因此包含在其Inclusive版本中的交易具有相同的安全保证。


### Fees and Rewards
Each transaction awards a fee to the creator of the ﬁrst block that included it in the set T.
Formally, let A be some block in sub(B, G).
Denote by T(A) the set of transactions which block A was the ﬁrst to contain, according to the order ≺<sub>B</sub> .
Then (according to B’s world view) A’s creator is awarded a fraction of the fee from every tx ∈ T(A).
Although naively we would want to grant A all of T(A)’s fees, security objectives cannot always permit it.
This is one of the main tradeoﬀs in the protocol: On the one hand, we wish to award fees to anyone that included a new transaction.
This implies that poorly connected miners that were slow to publish their block will still receive rewards.
On the other hand, oﬀ-chain blocks may also be the result of malicious action, including published blocks from a failed double-spend attack.
In this case we would prefer no payoﬀ would be received.
We therefore allow for a somewhat tolerant payment mechanism that grants a block A a fraction of the reward which depends on how quickly the block was referenced by the main chain.
The analysis that will follow (in Sect.
3) will justify the need for lower payments.


### 费用和奖励
每笔交易向包含在集合T中的第一个块的创建者收取费用。
正式地，让A为sub（B，G）中的某个块。
根据T <sub> B </sub>的顺序，用T（A）表示第一个包含块A的事务集。
然后（根据B的世界观），每个tx∈T（A）都会向A的创建者收取一小部分费用。
尽管我们天真地希望向A授予所有T（A）的费用，但是安全目标并不总是允许它。
这是该协议中的主要交易者之一：一方面，我们希望向包括新交易在内的任何人奖励费用。
这意味着那些关系不佳的矿工发布缓慢而仍会获得奖励。
另一方面，o-链块也可能是恶意行为的结果，包括来自双重支出攻击失败的已发布块。
在这种情况下，我们希望不会收到任何付款。
因此，我们允许某种程度宽容的支付机制，该机制向区块A授予一部分奖励，这取决于主链引用该区块的速度。
随后的分析（在第3节中）将证明需要较低的付款。


Formally, for any block A ∈ G deﬁne by pre(A) the latest main chain block which is reachable from A, and by post(A) the earliest main chain block from which A is reachable; if no such block exists, regard post(A) as a “virtual block” with height inﬁnity; if A is in the main chain then pre(A) = post(A) = A.
Denote c(A) := post(A).
height − pre(A).
height; c(·) is a measure of the delay in a block’s publication (with respect to the main chain).


形式上，对于任何一个A∈G块，都由pre（A）定义了从A可以到达的最新主链块，并由post（A）定义了从中可以到达A的最早主链块。
如果不存在这样的障碍物，则将桩（A）视为高度有限的“虚拟障碍物”； 如果A在主链中，则pre（A）= post（A）=A。
表示c（A）：= post（A）.
height-pre（A）.
height; c（·）是对块发布（相对于主链）的延迟的度量。


In order to penalize a block according to its gap parameter c(·) we make use of a generic discount function, denoted γ, which satisﬁes: γ : N ∪ {0} → [0, 1], it is weakly decreasing, and γ(0) = 1.
The payment for (the creator of) block A is deﬁned by:

为了根据块的间隙参数c（·）对其进行惩罚，我们使用通用折扣函数γ表示，该函数满足：γ：N∪{0}→[0，1]，它弱地减小，并且 γ（0）=1。
块A（的创建者）的付款由下式定义：

<img src="https://latex.codecogs.com/svg.latex?%5Cgamma%20%28c%28A%29%29%20%5Ccdot%20%5Csum_%7Bw%20%5Cepsilon%20T%28A%29%29%7D%20%7Bv%28w%29%7D" 
title="\gamma (c(A)) \cdot  \sum_{w \epsilon T(A))} {v(w)}" />

where v(w) is the fee of transaction w.
In other words, A gains only a fraction γ(c(A)) of its original rewards.
By way of illustration, consider the following discount function:

其中v（w）是交易费用w。
换句话说，A仅获得其原始奖励的分数γ（c（A））。
作为说明，请考虑以下折扣函数：

### Example 1.


<img src="https://latex.codecogs.com/svg.latex?%5Cgamma%20_0%20%28c%29%20%3D%5Cleft%5C%7B%5Cbegin%7Bmatrix%7D%201%20%26%200%5Cleq%20c%5Cleq%203%5C%5C%20%5Cfrac%7B10-c%7D%7B7%7D%20%26%203%20%3C%20c%20%3C%2010%20%26%20%281%29%5C%5C%200%20%26%20c%5Cgeq%2010%20%5Cend%7Bmatrix%7D%5Cright." 
title="\gamma _0 (c) =\left\{\begin{matrix} 1 & 0\leq c\leq 3\\ \frac{10-c}{7} & 3 < c < 10 & (1) \\ 0 & c\geq 10 \end{matrix}\right." />

γ<sub>0</sub> grants a full reward to blocks which are adequately synchronized with the main chain (γ<sub>0</sub> (c) = 1 for c ≤ 3), on the one hand, and pays no reward at all to blocks that were left “unseen” by the main chain for too long, on the other hand (γ<sub>0</sub> (c) = 0 for c ≥ 10); in the mid-range, a block is given some fraction of the transaction rewards (γ<sub>0</sub> (c) = (10−c)/7 for 3 < c < 10).


一方面，γ<sub> 0 </sub>向与主链充分同步的块（γ<sub> 0 </sub>（c）= 1，c≤3）授予完全奖励，并且 另一方面，对于由主链“看不见”太长时间的块完全不给予任何奖励（对于c≥10，γ<sub> 0 </sub>（c）= 0）； 在中间范围内，一个区块被给予一定比例的交易奖励（对于3<c<10，γ<sub>0</sub>(c)=（10-c）/ 7）。


### Money Creation 
In addition to fees, Bitcoin and other cryptocurrencies use the block creation process to create and distribute new coins.
Newly minted coins can also be awarded to oﬀ-chain blocks in a similar fashion to transaction fees, i.e., in amounts that decrease for blocks that were not quickly included in the main chain.
A block’s reward can therefore be set as a fraction γ(c(A)) of the full reward on the chain.
<sup>9</sup>  As our primary focus is on the choice of transactions

> <sup>9</sup> The total reward can be automatically adjusted to maintain a desired rate of money creation by a process similar to the re-targeting done for diﬃculty adjustments.


### 金钱创造
除了费用外，比特币和其他加密货币还使用区块创建过程来创建和分发新硬币。
新铸造的硬币也可以类似于交易费用的方式授予o型链块，即对于未迅速纳入主链的块减少的金额。
因此，可以将区块的奖励设置为链上全部奖励的分数γ（c（A））。
<sup> 9 </sup>因为我们的主要重点是选择交易

> <sup> 9 </sup>可以通过类似于困难调整的重新定位的过程来自动调整总奖励，以维持所需的货币创造率

to include in the block, we assume for simplicity from this point on, that no money creation takes place (i.e., that money creation has decayed to negligible amounts—as will eventually occur for Bitcoin).


为了简单起见，从现在开始，为了简单起见，我们假设没有货币创造发生（即，货币创造已衰减到可以忽略的数量，最终将发生在比特币中）。


Now that we have deﬁned the Inclusive protocol, we begin to analyze its implications.


现在我们已经定义了Inclusive协议，我们开始分析其含义。


# References
[10].Nisan, N., Roughgarden, T., Tardos, E., Vazirani, V.V.: Algorithmic game theory, chap.9.Cambridge University Press (2007)