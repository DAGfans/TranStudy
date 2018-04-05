# 3. FORMAL MODEL AND STATEMENT
# 3. 形式模型和陈述

In this section we describe our formal framework. 
While we introduce new notation and terminology, the reader should keep in mind that _we stick to Bitcoin’s model in almost every respect_—transactions, blocks, Proof-of-work, computationally bounded attacker, P2P propagation
of blocks, probabilistic security guarantees, etc. 
The “only” difference is that a block references (possibly) several predecessors rather than a single one. 
While this has far reaching consequences on how the ledger is to be interpreted, on the mining side things remain largely the same.

在本节中，我们将描述我们的形式框架。 
当我们引入新的符号和术语时，读者应该记住，_我们几乎在所有方面都遵守比特币模型_ - 交易，块，工作证明，计算能力有限的攻击者，块的点对点传播，有概率的安全保证等。
 “唯一”区别在于块（可能）引用了几个前序块而不是单个引用。 
 虽然这对账本如何解读具有深远的影响，但在挖矿方面，情况基本保持不变。


## A. Network
## A. 网络

We follow the model speciﬁed in [8]. 
The network of nodes (or miners) is denoted N , honest denotes the set of nodes that abide to the mining protocol (as deﬁned below), and malicious denotes the rest of the nodes. 
Honest nodes form a connected component in N ’s topology, and the communication delay diameter of the honest subnetwork is D: if an honest node $v ∈ N$ sends a message of size b MB at time t, it arrives at all honest nodes by time t + D the latest. 
The attacker is assumed to suffer no delays whatsoever on its outgoing or incoming links.

我们遵循[8]中指定的模型。
节点（或矿工）的网络表示为N，诚实节点（honest）表示遵守挖矿协议（如下定义）的节点集合，而作恶节点（malicious）表示其余节点。
诚实节点在N的拓扑结构中形成一个连通的组件，诚实子网络的通信延迟直径为D：如果一个诚实节点$v∈N$在时间t时发送了一个大小为b MB的消息，它最晚以时间t + D到达所有诚实节点。
攻击者被假设在其输出或输入链路上不会有任何延迟。 

The real value of D is a priori unknown. 
The PHANTOM protocol assumes that D is always smaller than some constant $D_{max}$(both depend on the block size b). 
The parameter $D_{max}$ is not hard-coded explicitly in the protocol, rather it influences another parameter,$k = k(D_{max})$, which is hard-coded and decided once and for all at the inception of the system. 
Roughly speaking, $k(D_{max})$ represents an upper bound on the number bound on the number of blocks that the network creates in one unit of delay and that may not be referenced by one another. 
Section 4 discusses this parameter in more detail.

D的真实值是无法预知的。 
PHANTOM协议假定D总是小于某个常数$D_{max}$（两者都取决于块大小b）。
参数$D_{max}$不是在协议中明确硬编码的，而是影响另一个参数，$k = k(D_{max})$，其在系统一初始时就是硬编码的并且会一直保持不变。
粗略地说，$k(D_{max})$代表网络在一个延迟单元中创建的块的数量上限，并且这些块彼此不会引用。
第4节更详细地讨论了这个参数。


## B. Mining framework
## B. 挖矿架构

**Proof-of-work.** Nodes create blocks of transactions by solving Proof-of-work puzzles. 
Block creation follows a Poisson process with parameter λ. 
For the sake of simplicity, we assume that λ is constant.^7 
The computational power of node $v∈ N$ is captured by $0 < αv< 1$ , which represents the probability that node v will be the creator of the next block in the system (at any point in time; this is a memoryless process). 
The attackers’ computational power is less than 50%. 
Thus $∑_{v∈N}αv= 1$, and $∑_{v∈malicious}αv := α <0.5$.

工作量证明。节点通过解决工作量证明难题来创建交易块。 
块创建遵循具有参数λ的泊松过程。 
为了简单起见，我们假设λ是恒定的。^7 
节点$v∈ N$的计算能力限定在$0 < α_v< 1$，它表示节点v将是系统中下一个块的创建者（在任何时间点; 这是一个无记忆的过程）的概率。 
攻击者的计算能力不到50％。 
因此，$∑_{v∈N}α_v= 1$， $∑_{v∈malicious}α_v := α <0.5$（译注：原文在这里用的符号=:是笔误，实际应该是:=，表示“定义为”）。

(^7) In practice, λ must occasionally be readjusted to account for shifting network conditions. 
PHANTOM can support a retargeting mechanism similar to Bitcoin’s, e.g., readjust every time that $Chn(G)$ grows by 2016 blocks. 

(^7) 在实践中，λ必须时不时重新调整以应对网络状况的变化。 
PHANTOM可以支持类似于比特币的重新定位机制，例如，每当$Chn(G)$ 增长2016个区块时进行重新调整。


**Block references.** Every block specifies its direct predecessors by referencing their ID in its header (a block’s ID is obtained by applying a collision resistant hash to its header);
 the choice of predecessors will be described in the next subsection. 
 This results in a structure of a direct acyclic graph (DAG) of blocks (as blocks can only reference blocks created before them), denoted typically $G= (C,E)$. 
 Here, C represents blocks and E represents the hash references. 
 We will frequently write $B∈G$ instead of $B∈C$.

**区块引用。** 每个块通过在它的头部中引用它的直接前序块的ID来指定前序块（块的ID是通过对其头部执行抗碰撞散列来获得的）； 
前序块的选择将在下一小节中介绍。 
这就形成了块的有向无环图（DAG）的结构（因为块只能引用在其之前创建的块），通常表示为$G= (C,E)$。 
这里，C表示块，E表示散列引用。 
我们会经常写成 $B∈G$ 而不是 $B∈C$。

**DAG topology.** The topology of the blockDAG induces a natural partial ordering over blocks,
as follows: if there is a path in the DAG from block C to block B we write $B∈past(C)$; 
in this case, C was provably created after B and therefore B should precede C in the order.^8 
A node does not consider a block as valid until it receives its entire past set. 
The unique block genesis is the block created at the inception of the system, and every valid block must have it
in its past set.

**DAG 结构.** 区块有向无环图（BlockDAG）的结构会导致块天然的偏序，如下所示：如果DAG中存在从块C到块B的路径，表示为$B∈past(C)$; 
在这种情况下，C很大可能是在B之后创建的，因此B应该按照顺序在C之前。^8
一个节点只有在接收到块的整个过去集时才认为该块有效的。 
唯一的创始区块是在系统开始时创建的，每个有效的区块都必须在其过去集中包含它。

Similarly, the future set of a block, $future(B)$, represents blocks that were provably created after it:
$B ∈ past (C) \Leftrightarrow  C ∈ future (B)$.
In contrast to the past set, the future set of a block keeps growing in time, as more blocks are created and are referencing it.
To avoid ambiguity, we write $future(B)∩G$ or $future(B,G)$, and write $future(B)$ only when the context is clear or unimportant.

类似地，一个区块的未来集 $future(B)$ 代表很大可能在它之后创建的块：$B ∈ past (C) \Leftrightarrow  C ∈ future (B)$。 
与过去集相反，随着更多的块被创建并引用它，一个区块的未来集会不断增加。 
为了避免歧义，我们写成$future(B)∩G$或$future(B,G)$，而只有在上下文明确或者不重要时才写成$future(B)$。

The set $anticone (B)$ represents all blocks not in B ’s future or past (excluding B as well). 
These are blocks whose ordering with respect to B is not deﬁned via the partial ordering that the topology of the DAG induces. 
Formally, for two distinct blocks $B, C ∈ G: C ∈ anticone (B, G) \Leftrightarrow (B \notin past (C) ∧ C \notin past (B)) \Leftrightarrow B ∈ anticone (C, g)$. 
Here too we usually specify the context, $anticone (B, G)$, because the anticone set can grow with time.

集合$anticone (B)$表示所有不在B的未来集或过去集的块（也不包括B）。 
这些块与B之间的顺序无法通过已有的DAG拓扑结构的偏序来确定。
形式上，对于两个不同的块$B, C ∈ G: C ∈ anticone (B, G) \Leftrightarrow (B \notin past (C) ∧ C \notin past (B)) \Leftrightarrow B ∈ anticone (C, g)$.。
这里我们也通常需要指定上下文，$anticone (B, G)$，因为anticone集（反锥体集）会随时间增长。

In Figure 1 above we illustrates this terminology.

在上面的图1中，我们演示了这个术语。

(^8) Note that an edge in the DAG points back in time, from the new block to previously created blocks which it references.

(^8) 请注意，DAG中一条边会及时从新块指向到它引用的之前创建的块上。

**DAG mining protocol.** $G_t^v$ denotes the block DAG that node $v ∈ N$ observes at time t. 
This DAG represents the history of all (valid) block-messages received by the node. 
$G_t^{oracle} := ∪_{v∈N} G_t^v$ denotes the block DAG of a hypothetical oracle node, and $G_t^{pub} := ∪_{v∈honest} G_t^v$ denotes the block DAG containing all blocks that are visible to some honest node(s).

**DAG 挖矿协议.** $G_t^v$表示节点$v ∈ N$在时间t观察到的区块有向无环图（BlockDAG）。 
这个有向无环图（DAG）表示节点接收到的所有（有效）块消息的历史。
$G_t^{oracle} := ∪_{v∈N} G_t^v$表示某个假定的预言节点的区块有向无环图（BlockDAG），而$G_t^{pub} := ∪_{v∈honest} G_t^v$表示包含了对某个(些)诚实节点可见的所有区块的有向无环图(BlockDAG)。
**译注：** 这个oracle在Spectre论文应用比较多，是一个假定拥有所有信息的节点，方便论述

A tip of the DAG is a leaf-block, namely, a block with in-degree 0. 
The instructions to a miner in the DAG paradigm are simple:

DAG的末梢是一个叶块，即一个入度为0的块。
在有向无环图（DAG）范例中对矿工的规则很简单：

1. When creating or receiving a block, transmit it to all of one’s peers in N . Formally, this implies that $∀v, u ∈ honest : G_t^v ⊆ G_{t+D}^u$ .
2. When creating a block, embed in its header a list containing the hash of all tips in the locally-observed DAG. Formally, this implies that if block B was created at time t, by honest node v , then $past (B) = G_t^v$ . ^9

Since these are the only two mining rules in our system, a byzantine behaviour of the attacker (which controls up to α of the mining power) amounts to an arbitrary deviation from one or both of these instructions.


1.	当某个节点创建或接收一个块时，将该块发送给网络N中的该节点的所有对等节点。形式上，这意味着：$∀v, u ∈ honest : G_t^v ⊆ G_{t+D}^u$ 。
2.	在创建块时，在其头部中嵌入一个包含本地观察到的DAG中所有末梢的散列列表。 在形式上，这意味着如果块B在时间t处由诚实节点v创建，则B的过去集$past (B) = G_t^v$。^9

由于这些是我们系统中唯一的两个挖矿规则，所以攻击者的拜占庭行为（控制达到占比为α的挖矿能力）至少违反了其中一条规则。

(^9) Technically it is more accurate to write $past(B) =G_v^t \setminus {B}$, as a block does not belongs to its own past set.
(^9) 从技术上讲，写成$past(B) =G_v^t \setminus {B}$会更准确，因为一个区块不属于它自己的过去集。


## C. DAG client protocol 
## C. DAG 客户端协议

The DAG as described so far possibly embeds conﬂicting transactions. 
These are resolved on the client level.
A client can be deﬁned formally as a node in N which has no mining power. 
Intuitively, it is any user of the system who is interested in reading and interpreting the current state of the ledger.

目前为止所描述的DAG可能会嵌入冲突交易。 
这些都在客户端解决。
客户端可以正式定义为N中没有挖矿能力的节点。 
直观地说，它是系统中任何对解读账本当前状态有兴趣的用户。

In this work, a transaction tx is an arbitrary message that is embedded in a block. 
An underlying Consistency rule takes as input a set T of transactions and returns valid or invalid. 
Our work is agnostic to the deﬁnition and operation of this rule, or to the characterization of the transaction space.
Instead, we focus on the following task: devising a protocol through which all nodes agree on the order of all transactions in the system. 
Once such an order is agreed, one can iterate over all transactions, in the prescribed order, and approve each transaction that is consistent – according to the underlying rule – with those approved so far. 
Such an ordering rule constitutes the client protocol, and is run by each client locally without any need to communicate additional messages with other clients.

在这项工作中，交易tx是嵌入在块中的任意消息。 
一个底层的一致性规则将输入交易集合T并返回有效或无效。 
我们的工作对于这个规则的定义和运行，或对于交易所属领域的信息都是不可知的。 
相反，我们专注于以下任务：设计一个协议，所有节点通过该协议就系统中所有交易的排序达成一致。 
这种排序一旦达成，就可以按照规定的顺序对所有交易进行遍历，并根据底层规则确认每笔与当前已确认的交易一致的交易。
这种排序规则构成了用户协议，并且由本地客户端运行，无需与其他客户端通信额外的消息。

Formally, an ordering rule $ord$ takes as input a blockDAG G and outputs a linear order over G’s blocks, $ord(G) = (B_0 , B_1 , . . . , B_{|G|} )$. 
Transactions in the same block are ordered according to their appearance in it, and this convention allows us to talk henceforth on the order of blocks only. 
With respect to a given rule $ord$, we write $B \prec_{ord(G)}C$ if the index of B precedes that of C in $ord(G)$; 
we abbreviate and write $B\prec_GC$ or even $B\prec C$ when the context is understood. 
For convenience, we use the same notation $B\prec_GC$ when $B ∈ G$ but $C\notin G$.

形式上，排序规则$ord$将块DAG G作为输入并输出G块上的线性排序，$ord(G) = (B_0 , B_1 , . . . , B_{|G|} )$.  
在同一个块中的交易根据它们出现先后进行排序，因此这个约定允许我们今后只讨论块的顺序。 
关于给定的规则$ord$，如果B的索引在$ord(G)$中的C的索引之前，我们写$B \prec_{ord(G)}C$; 
我们在能上下文时缩写成$B\prec_GC$，甚至是$B\prec C$。 
为了方便起见，当$B ∈ G$ 但 $C\notin G$时，我们使用相同的符号$B\prec_GC$。

## D. Convergence of the order
## D. 顺序的收敛

The following definition captures the desired security of the protocol, in terms of the
probability that some order between two blocks will be reversed.

下面的定义通过两个块之间的顺序会被颠倒的概率，约束了协议所要求的安全性。

**Deﬁnition 3.** Fix a rule $ord$. 
Let $B ∈ G = G_t^{pub}$ . 
The function Risk is deﬁned by the probability that a block that did not precede B in time $t_1 ≥ t_0$ will later come to precede it: 
$Risk(B, t_1):= Pr \left ( ∃s>t_1,∃C∈G_s^{pub}: B\prec_{G_t^{pub}}C∧C\prec_{G_s^{pub}}B \right )$ .

定义3. 规定一个规则ord。 
设 $B ∈ G = G_t^{pub}$. 
函数Risk定义为一个没有在B之前的块在时间$t_1 ≥ t_0$后会出现在B之前的概率：
$Risk(B, t_1):= Pr \left ( ∃s>t_1,∃C∈G_s^{pub}: B\prec_{G_t^{pub}}C∧C\prec_{G_s^{pub}}B \right )$ .
（译注：∃ 表示“存在”——即英文Exist的意思。）

In the deﬁnition above, the probability is taken over all random events in the network, including block creation and propagation, as well as the attacker’s arbitrary (byzantine) behaviour. 
The convergence property below guarantees that the order between a block and those succeeding it, or those not published yet, will not be reversed, w.h.p. 
This captures the security of the protocol, as it provides honest nodes with (probabilistic) security guarantees regarding possible reorgs.

在上面的定义中，网络中的所有随机事件，包括块创建和传播以及攻击者的任意（拜占庭）行为是有概率的。 
下面的收敛属性尽可能保证了一个区块和它后继区块或尚未发布的区块之间的顺序不会被颠倒。 
这些限定了协议的安全性，因为它在概率上为诚实的节点提供了关于可能的重组的安全保证。


**Property 1.** An ordering rule $ord$ is converging if $∀t_0 > 0$ and $B ∈ G_t^{pub} :\lim_{t_1 \to ∞} Risk (B, t_1 ) = 1$, even when a fraction α of the mining power is byzantine.

**特性1.** 如果$∀t_0 > 0$ 并且 $B ∈ G_t^{pub} :\lim_{t_1 \to ∞} Risk (B, t_1 ) = 1$，那么即使占比α的算力出现拜占庭行为时，排序规则$ord$也是收敛的。

**Remark.** Property 1 essentially couples the Safety and Liveness properties required from consensus protocols. Indeed, once $Risk( B , t_1)< \epsilon$ , a decision to accept transactions in B can be made (Liveness), and is guaranteed to be irreversible (Safety) up to an error probability of $\epsilon$  (as in Bitcoin and other protocols). 
Nevertheless, we avoid phrasing our results in these terms, for the sake of clarity of presentation. 
The complication arises from the need to analyze the system from the perspective of every node $G_t^v$ , and not merely from the public ledger’s hypothetical perspective $G_t^{pub}$ ; 
this technicality is not unique to PHANTOM, and should be regarded in any work that formalizes blockchain based consensus (unless propagation delays are assumed to be negligible). 
We leave the task of bridging this gap to a later version.

**备注**  特性1实质上结合了共识协议所要求的安全性和活性。
事实上，一旦$Risk( B , t_1)< \epsilon$，就可以确认B中交易（活性），并且保证是不可逆的（安全），直到错误概率为 $\epsilon$（如比特币和其他协议）。
尽管如此，为了清楚，我们避免用这些术语来表述我们的结果。 
复杂性的产生源于需要从每个节点$G_t^v$的角度分析系统，而不仅仅是从公共帐本假设的角度$G_t^{pub}$ 出发; 这种技术性问题对于PHANTOM来说并不是独一无二的，并且应该在形式化任何基于区块链的共识的工作中予以考虑（除非假定传播延迟可以忽略不计）。 
我们将降低复杂性的任务留到之后的版本。

The security threshold is the minimal hashing power that the attacker must acquire in order to disrupt the protocol’s operation:

安全阈值是攻击者为了破坏协议操作而必须获得的最小算力：

**Deﬁnition 4.** The security threshold of an ordering rule $ord$ is deﬁned as the maximal α (attacker’s relative computational power) for which Property 1 holds true.

**定义4** 排序规则$ord$的安全阈值定义为特性1为真时的最大α（攻击者的相对计算能力）。

A protocol is scalable if it is safe to increase the block creation rate λ without compromising the security, that is, if the security threshold does not deteriorate as λ increases (this can be phrased also in terms of increasing the block size b rather than λ).

如果在不影响安全性的情况下增加区块创建率λ是安全的，则协议是可扩展的，即，如果安全阈值不随着λ增加而变差（这也可以用增加块大小b而不是λ来表示）。


## E. Main result
## E. 主要结果

Our goal in this paper is to describe formally the ordering procedure of PHANTOM and to prove that it is scalable in the above sense.

我们在本文中的目标是形式化地描述PHANTOM的排序过程并证明它在上述条件下是可扩展的。

**Theorem 5**  (PHANTOM scales). Given a block creation rate $λ > 0, δ > 0$, and $D_{max} > 0$, if $D_{max}$ is equal to or greater than the network’s propagation delay diameter D, then the security threshold of PHANTOM, parameterized with $k(D_{max} , δ)$, is at least $1/2 \cdot (1-\delta )$.

**定理5** (PHANTOM可扩展). 给定块创建率$λ > 0, δ > 0$且$D_{max} > 0$，如果$D_{max}$大于或等于网络的传播延迟直径D，则以$k(D_{max} , δ)$作为参数的PHANTOM的安全阈值至少为$1/2 \cdot (1-\delta )$。

The parameterization of PHANTOM via $k(D_{max} , δ)$ is deﬁned in the subsequent section (see (1)). 
Theorem 5 encapsulates the main achievement of our work. 
We prove the theorem formally in Section 5. 
Contrast this result to a theorem regarding the Bitcoin protocol, which appears in several forms in previous work (e.g., [6], [9]):

PHANTOM通过$k(D_{max} , δ)$的参数化在后面的章节中定义(参见(1))。 
定理5包括了我们工作的主要成果。 
我们在第5节中正式证明了该定理。
该结果与比特币协议的定理相比较，在过去的工作中已经以几种形式出现(例如，[6]，[9])：

**Theorem 6** (Bitcoin does not scale). As λ increases, the security threshold of the Bitcoin protocol goes to 0.

**定理6** (比特币不可扩展)。随着λ增加，比特币协议的安全阈值变为0。

Finally, we note that even if $D_{max} \ngeqslant D$, the system’s security does not immediately break apart. Rather, the minimal power needed to attack the system goes from 50% to 0, deteriorating at a rate that depends on the error gap $D-D_{max}$ .

最后，我们注意到，即使$D_{max} \ngeqslant D$，系统的安全性并没有立即崩溃。
而是，攻击系统所需的最小算力需从50%降低到0，退化速率取决于误差$D-D_{max}$。

