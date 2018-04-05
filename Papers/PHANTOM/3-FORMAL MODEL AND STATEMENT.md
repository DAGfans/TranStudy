#### 3. FORMAL MODEL AND STATEMENT
#### 3. 形式模型和陈述

In this section we describe our formal framework. While we introduce new notation and
terminology, the reader should keep in mind thatwe stick to Bitcoin’s model in almost every
respect—transactions, blocks, Proof-of-work, computationally bounded attacker, P2P propagation
of blocks, probabilistic security guarantees, etc. The “only” difference is that a block references
(possibly) several predecessors rather than a single one. While this has far reaching consequences
on how the ledger is to be interpreted, on the mining side things remain largely the same.

在本节中，我们将描述我们的正式框架。 当我们引入新的符号和术语时，读者应该记住，我们几乎在所有方面都遵守比特币模型 - 交易，块，工作证明，计算有限的攻击者，块的点对点传播，概率安全保证等。 “唯一”区别在于块（可能）引用了几个前序块而不是单个引用。 虽然这对分类账如何解读具有深远的影响，但在挖矿方面，情况基本保持不变。


A. Network

We follow the model specified in [8]. The network of nodes (or miners) is denotedN,honest
denotes the set of nodes that abide to the mining protocol (as defined below), andmalicious
denotes the rest of the nodes. Honest nodes form a connected component in N’s topology, and
the communication delay diameter of the honest subnetwork is D: if an honest nodev∈ N
sends a message of size b MB at timet, it arrives at all honest nodes by time t+D the latest.
The attacker is assumed to suffer no delays whatsoever on its outgoing or incoming links.

The real value of D is a priori unknown. The PHANTOM protocol assumes that D is always
smaller than some constant Dmax(both depend on the block sizeb). The parameter Dmax is not
hard-coded explicitly in the protocol, rather it influences another parameter,k=k(Dmax), which
is hard-coded and decided once and for all at the inception of the system. Roughly speaking,
k(Dmax) represents an upper bound on the number bound on the number of blocks that the
network creates in one unit of delay and that may not be referenced by one another. Section 4
discusses this parameter in more detail.

我们遵循[8]中指定的模型。节点（或矿工）的网络表示为N，诚实者（honest）表示遵守采矿协议（如下定义）的节点集合，而作恶者（malicious）表示其余节点。诚实节点在N的拓扑结构中形成一个连通组件，诚实子网络的通信延迟直径为D：如果一个诚实节点v∈N在时间t时发送了一个大小为b MB的消息，它最晚以时间t + D到达所有诚实节点。假设攻击者在其输出或输入链路上不会有任何延迟。 

D的真正价值在于先前未知。 PHANTOM协议假定D总是小于某个常数Dmax（两者都取决于块大小b）。参数Dmax不是在协议中明确硬编码的，而是影响另一个参数，k = k（Dmax），而其在系统一开始就是硬编码的并且一劳永逸。粗略地说，k（Dmax）代表网络在一个延迟单元中创建的块的数量上绑定的数量上限，并且可能不被彼此引用。第4节更详细地讨论了这个参数。


B. Mining framework
B. 挖矿架构

Proof-of-work.Nodes create blocks of transactions by solving Proof-of-work puzzles. Block creation follows a Poisson process with parameterλ. For the sake of simplicity, we assume that λ is constant.^7 The computational power of nodev∈ Nis captured by 0 < αv< 1 , which represents the probability that nodevwill be the creator of the next block in the system (at any point in time; this is a memoryless process). The attackers’ computational power is less than 50%. Thus ∑v∈Nαv= 1, and ∑v∈maliciousαv := α <^0.^5.

工作证明。节点通过解决工作证明难题来创建交易块。 块创建遵循具有参数λ的泊松过程。 为了简单起见，我们假设Λ是恒定的。^ 7 由0 <αv<1捕获节点v∈N的计算能力，它表示节点v将是系统中下一个块的创建者（在任何时间点; 这是一个无记忆的过程）的概率。 攻击者的计算能力不到50％。 因此，Σv∈Nαv= 1，Σv∈malicious αv := α<0.5（译注：原文在这里用的符号=:是笔误，实际应该是:=，表示“定义为”）。

(^6) Or, tx ̄can appear in B before tx, but this is not an interesting scenario.
(^7) In practice, λ must occasionally be readjusted to account for shifting network conditions. PHANTOM can support a retargeting mechanism similar to Bitcoin’s, e.g., readjust every time that Chn(G) grows by 2016 blocks. 

(^6) 或者，tx ̄可以在tx之前出现在B中，但这不是一个有趣的场景。
(^7) 在实践中，λ必须偶尔重新调整以应对网络状况的变化。 PHANTOM可以支持类似于比特币的重新定位机制，例如，每当Chn(G)增长2016个区块时进行重新调整。


Block references.Every block specifies its direct predecessors by referencing their ID in its
header (a block’s ID is obtained by applying a collision resistant hash to its header); the choice
of predecessors will be described in the next subsection. This results in a structure of a direct
acyclic graph (DAG) of blocks (as blocks can only reference blocks created before them), denoted
typicallyG= (C,E). Here,Crepresents blocks andErepresents the hash references. We will
frequently writeB∈Ginstead of B∈C.

区块引用。 每个块通过引用它们在其头部中的ID来指定它的直接前序块（块的ID是通过对其头部应用抗碰撞散列来获得的）。 前序块的选择将在下一小节中介绍。 这就形成了块的有向无环图（DAG）的结构（因为块只能引用在其之前创建的块），通常表示为G =（C，E）。 这里，C表示块，E表示散列引用。 我们会经常写B∈G而不是B∈C。

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

The set anticone(B) represents all blocks not inB’s future or past (excludingBas well).
These are blocks whose ordering with respect to B is not defined via the partial ordering
that the topology of the DAG induces. Formally, for two distinct blocks B,C ∈ G:C ∈
anticone(B,G) ⇐⇒ (B /∈past(C)∧C /∈past(B)) ⇐⇒ B∈anticone(C,g). Here too
we usually specify the context,anticone(B,G), because the anticone set can grow with time.

In Figure 1 above we illustrates this terminology.

区块有向无环图（BlockDAG）的拓扑结构导致块的自然偏序，如下所示：如果DAG中存在从块C到块B的路径，则写入B∈past（C）; 在这种情况下，C可证明是在B之后创建的，因此B应该按照顺序在C之前。^ 8一个节点在接收到它的整个过去集之前并不认为它是有效的。 独特的块 创始区块 生成是系统开始时创建的，每个有效的区块都必须在其过去集中包含它。

类似地，一个区块的未来集 future(B) 代表可证明在它之后创建的块：B∈past（C）⇐⇒ C∈ future(B)。 与过去集相反，随着更多的块被创建并引用它，一个区块的未来集会不断增加。 为了避免歧义，我们只在上下文清晰或不重要时才写做future(B)∩G或future(B, G)，和future(B)。

集合anticone（B）表示所有不在B的未来集或过去集的块（也不包括B）。 这些块关于B的排序没有通过DAG拓扑引起的偏序排列来定义。 形式上，对于两个不同的块B,C ∈ G: C ∈ anticone(B,G) ⇐⇒ (B /∈past(C)∧C /∈past(B)) ⇐⇒ B∈anticone(C,g).。 这里我们也通常指定上下文，anticone(B,G)，因为anticone集（反锥体集）可以随时间增长。

在上面的图1中，我们演示了这个术语。

(^8) Note that an edge in the DAG points back in time, from the new block to previously created blocks which it references.
(^8) 请注意，DAG中一个边缘会及时从新块指向到它引用的之前创建的块上。

DAG mining protocol.Gvt denotes the block DAG that nodev∈Nobserves at timet. This DAG
represents the history of all (valid) block-messages received by the node. Goraclet :=∪v∈NGvt
denotes the block DAG of a hypothetical oracle node, andGpubt :=∪v∈honestGvt denotes the
block DAG containing all blocks that are visible to some honest node(s).
Atipof the DAG is a leaf-block, namely, a block with in-degree 0. The instructions to a
miner in the DAG paradigm are simple:


1) When creating or receiving a block, transmit it to all of one’s peers inN. Formally, this
implies that∀v,u∈honest:Gvt⊆Gut+D.

2) When creating a block, embed in its header a list containing the hash of all tips in the
locally-observed DAG. Formally, this implies that if block B was created at timet, by
honest nodev, thenpast(B) =Gvt.^9

Since these are the only two mining rules in our system, a byzantine behaviour of the attacker
(which controls up toαof the mining power) amounts to an arbitrary deviation from one or
both of these instructions.

DAG 挖矿协议. Gvt表示节点v∈N在时间t观察到的区块有向无环图（BlockDAG）。 这个有向无环图（DAG）表示节点接收到的所有（有效）块消息的历史。Goracle t :=∪v∈NGvt表示假定的预言节点的区块有向无环图（BlockDAG），而Gpubt :=∪v∈honestGvt表示包含了对某个(些)诚实节点可见的所有区块的有向无环图(BlockDAG)。
DAG的末梢是一个叶块，即一个入度为0的块。在有向无环图（DAG）范例中对矿工的说明很简单：

1.	当创建或接收一个块时，将其发送给N中的所有对等者。形式上，这意味着：∀v,u∈honest: Gvt⊆Gut+D 。
2.	在创建块时，在其头部中嵌入一个包含本地观察到的DAG中所有末梢的散列列表。 在形式上，这意味着如果块B在时间t处由诚实节点v创建，则B的过去集past（B）= Gvt。^9

由于这些是我们系统中唯一的两个挖矿规则，所以攻击者的拜占庭行为（控制达到α 采矿能力）相当于这一两条说明种肆意的违背。

(^9) Technically it is more accurate to write past(B) =Gvt{B}, as a block does not belongs to its own past set.
(^9) 从技术上讲，写成past(B) =Gvt{B}会更准确，因为一个区块不属于它自己的过去集。


C. DAG client protocol
C. DAG 客户端协议

The DAG as described so far possibly embeds conflicting transactions. These are resolved on
the client level. Aclientcan be defined formally as a node inN which has no mining power.
Intuitively, it is any user of the system who is interested in reading and interpreting the current
state of the ledger.

In this work, a transactiontxis an arbitrary message that is embedded in a block. An
underlying Consistency rule takes as input a setTof transactions and returns valid or invalid.
Our work is agnostic to the definition and operation of this rule, or to the characterization of the
transaction space. Instead, we focus on the following task: devising a protocol through which
all nodes agree on the order of all transactions in the system. Once such an order is agreed, one
can iterate over all transactions, in the prescribed order, and approve each transaction that is
consistent – according to the underlying rule – with those approved so far.Such an ordering rule
constitutes the client protocol, and is run by each client locally without any need to communicate
additional messages with other clients.

Formally, an ordering rule ord takes as input a blockDAG G and outputs a linear order over
G’s blocks,ord(G) = (B 0 ,B 1 ,...,B|G|). Transactions in the same block are ordered according
to their appearance in it, and this convention allows us to talk henceforth on the order of blocks
only. With respect to a given ruleord, we write B≺ord(G)C if the index of B precedes that of
C in ord(G); we abbreviate and write B≺GC or even B≺C when the context is understood.
For convenience, we use the same notation B≺GC when B∈G but C /∈G.

目前为止所描述的DAG可能会嵌入冲突交易。 这些都在客户端解决。 客户端可以正式定义为N中没有挖掘能力的节点。 直观地说，它是系统中任何对解读分类账当前状态都有兴趣的用户。

在这项工作中，交易tx是嵌入在块中的任意消息。 一个基础的一致性规则将输入一组T的交易并返回有效或无效。 我们的工作对于这个规则的定义和运行，或对于交易空间的描述都是不可知的。 相反，我们专注于以下任务：设计一个协议，所有节点通过该协议就系统中所有交易的排序达成一致。 这种排序一旦达成，就可以按照规定的顺序对所有交易进行迭代，并根据基本规则批准每个交易与迄今已批准的交易一致。这种排序规则构成了用户协议，并且由本地客户端运行，无需与其他客户端通信额外的消息。

形式上，排序规则ord将块DAG G作为输入并输出G块上的线性排序，ord(G) = (B 0 ,B 1 ,...,B|G|).  在同一个块中的交易根据它们出现先后进行排序，并且这个约定只允许我们今后根据块的顺序进行讨论。 关于给定的规则ord，如果B的索引在ord（G）中的C的索引之前，我们写B≺ ord（G）C; 我们在上下文被理解时缩写成B≺GC，甚至是B≺C。 为了方便起见，当B∈G但C /∈G时，我们使用相同的符号B≺GC。


D. Convergence of the order

D. 顺序的收敛

The following definition captures the desired security of the protocol, in terms of the
probability that some order between two blocks will be reversed.

下面的定义通过两个块之间的顺序会被颠倒的概率，描述了协议所要求的安全性。

Definition 3. Fix a rule ord. LetB∈G=Gpubt 0. The function Riskis defined by the probability
that a block that did not precedes Bin timet 1 ≥t 0 will later come to precede it:Risk(B,t 1 ) := Pr(∃s > t 1 ,∃C∈Gpubs :B≺Gpubt 1 C∧C≺Gpubs B).

定义3. 规定一个规则ord。 设B∈G= Gpubt 0. 函数Risk定义为在时间t1≥ t0 ，B之后出现的区块但优先于B的概率：Risk(B,t 1 ) := Pr(∃s > t 1 ,∃C∈Gpubs :B≺Gpubt 1 C∧C≺Gpubs B)。（译注：∃ 表示“存在”——即英文Exist的意思。）

In the definition above, the probability is taken over all random events in the network, including
block creation and propagation, as well as the attacker’s arbitrary (byzantine) behaviour. The
convergence property below guarantees that the order between a block and those succeeding it, or those not published yet, will not be reversed,w.h.p.This captures the security of the protocol, as it provides honest nodes with (probabilistic) security guarantees regarding possible reorgs.

在上面的定义中，网络中的所有随机事件，包括块创建和传播 以及攻击者的肆意（拜占庭）行为都会被接管。 下面的收敛属性尽可能保证了一个区块和它后继去块或尚未生成区块之间的顺序不会被颠倒。 这里抓住了协议的安全性，因为它在概率上为可靠的节点提供了关于可能重组的安全保证。


Property 1. An ordering rule ord is converging if ∀t0 > 0 and B ∈ G pub t0 : lim t1→∞ Risk (B, t1) = 0, even when a fraction α of the mining power is byzantine.

Remark. Property 1 essentially couples the Safety and Liveness properties required from consensus protocols. Indeed, once Risk (B, t1) < , a decision to accept transactions in B can be made (Liveness), and is guaranteed to be irreversible (Safety) up to an error probability of  (as in Bitcoin and other protocols). Nevertheless, we avoid phrasing our results in these terms, for the sake of clarity of presentation. The complication arises from the need to analyze the system from the perspective of every node Gv t , and not merely from the public ledger’s hypothetical perspective Gpubt ; this technicality is not unique to PHANTOM, and should be regarded in any work that formalizes blockchain based consensus (unless propagation delays are assumed to be negligible). We leave the task of bridging this gap to a later version.

The security threshold is the minimal hashing power that the attacker must acquire in order to
disrupt the protocol’s operation:

Definition 4.The security threshold of an ordering ruleord is defined as the maximalα
(attacker’s relative computational power) for which Property 1 holds true.

A protocol is scalable if it is safe to increase the block creation rateλwithout compromising
the security, that is, if the security threshold does not deteriorate asλincreases (this can be
phrased also in terms of increasing the block sizebrather thanλ).

特性1. 如果∀t0 > 0 and B ∈ G pub t0 : lim t1→∞ Risk (B, t1) = 0，那么即使当算力的一个分数α出现拜占庭行为时，排序规则ord也是收敛的。

备注。 特性1实质上结合了共识协议所要求的安全性和活跃性。事实上，一旦Risk (B, t1) < ，接受B中交易的决定可以被生成（活跃性），并且保证是不可逆的（安全），直到错误概率为（如比特币和其他协议）。 尽管如此，为了清楚表达，我们避免用这些术语来表述我们的结果。 复杂性的产生源于需要从每个节点Gvt的角度分析系统，而不仅仅是从公共帐本的假设角度出发; 这种技术性对于PHANTOM来说并不是独一无二的，并且应该在任何区块链共识合规的工作中予以考虑（除非假定传播延迟可以忽略不计）。 我们将弥合差距的任务留到之后的版本。

安全阈值是攻击者为了破坏协议操作而必须获得的最小算力：

定义4排序规则ord的安全阈值定义为特性1为真的最大α（攻击者的相对计算能力）。
如果在不影响安全性的情况下增加区块创建率λ是安全的，则协议是可扩展的；即，如果安全阈值不随着λ增加而变差（这也可以用增加块大小b而不是λ来表示）。


E. Main result
E. 主要结果

Our goal in this paper is to describe formally the ordering procedure of PHANTOM and to
prove that it is scalable in the above sense.

Theorem 5(PHANTOM scales).Given a block creation rate λ > 0 ,δ > 0 , and Dmax> 0 , if
Dmax is equal to or greater than the network’s propagation delay diameter D, then the security
threshold of PHANTOM, parameterized with k(Dmax,δ), is at least 1 / 2 ·(1−δ).

The parameterization of PHANTOM viak(Dmax,δ) is defined in the subsequent section
(see (1)). Theorem 5 encapsulates the main achievement of our work. We prove the theorem
formally in Section 5. Contrast this result to a theorem regarding the Bitcoin protocol, which
appears in several forms in previous work (e.g., [6], [9]):

Theorem 6(Bitcoin does not scale).Asλincreases, the security threshold of the Bitcoin protocol
goes to 0.

Finally, we note that even if D max6≥D, the system’s security does not immediately break
apart. Rather, the minimal power needed to attack the system goes from 50% to 0, deteriorating
at a rate that depends on the error gap D−Dmax.

我们在本文中的目标是正式描述PHANTOM的排序过程并证明它在上述意义上是可扩展的。

定理5(PHANTOM可扩展).给定块创建率λ> 0，δ> 0且Dmax> 0，如果Dmax等于或大于网络的传播延迟直径D，则用k（Dmax，δ）参数化的PHANTOM的安全阈值为至少为1/2·（1-δ）。
PHANTOM通过k（Dmax，δ）的参数化在后面的章节中定义(参见(1))。 定理5包括了我们工作的主要成果。 我们在第5节中正式证明了该定理。并将该结果与关于比特币协议的定理相比较，而其在过去的工作中以几种形式出现(例如，[6]，[9])：

定理6(比特币不可扩展)。随着λ增加，比特币协议的安全阈值变为0。
最后，我们注意到，即使Dmax 6≥ D，系统的安全性并没有立即分开。相反，攻击系统所需的最小算力需从50%降低到0，退化速率取决于误差间隙D−DMAX。

