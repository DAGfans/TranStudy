> **Source：** [https://eprint.iacr.org/2016/1159.pdf](https://eprint.iacr.org/2016/1159.pdf)  
> **TranStudy：** [https://github.com/DAGfans/TranStudy/new/master/Papers/SPECTRE](https://github.com/DAGfans/TranStudy/new/master/Papers/SPECTRE)

# 2. FORMAL STATEMENT OF THE PROBLEM
# 2. 问题的形式化论述

In this section we describe a generic framework for reasoning about the security and scalability properties of cryptocurrency protocols. 
Generally, in our framework, a cryptocurrency protocol speciﬁes two sets of instructions – the mining protocol, regarding the creation of blocks and formation of the block ledger, and the TxO protocol, interpreting the ledger and extracting from it a consistent subset of valid transactions. 
Since transactions in the protocol are accepted with increasing probability as time goes on, users additionally run the Robust TxO protocol, to quantify the robustness of an accepted transaction – a bound on the probability that it will ever be reversed, when a malicious attacker attempts to do so (Bitcoin transactions for example, can be reversed if the attacker manages to produce a longer alternative chain on which they are not present – this event occurs with decreasing probability as time passes). 
Next, we present our framework in the abstract sense, so as to keep it as general as possible. 
In Section 4, we present a protocol that meets the requirements and uses a block DAG to do so. 
We defer the speciﬁcs of the solution and the mining protocol, till then. 
We will use our framework to make clear the sense in which SPECTRE avoids the security-scalability trade-off that Bitcoin suffers from. 

在本节中，我们将介绍用于论述加密货币协议的安全性和可扩展性属性的通用框架。
通常，在我们的框架中，一个加密货币协议指定了两套指令 - 挖矿协议，与区块的创建和区块账本的形成有关，以及TxO协议，解释账本并从中提取一个一致的有效交易的子集。
由于协议中的交易随着时间的推移而越来越可能被接受，所以用户需要另外运行Robust TxO协议来量化已接受交易的健壮性 - 即当恶意攻击者试图攻击协议时，协议被逆转的可能性的界限（比如比特币交易，如果攻击者设法产生一个还未出现的更长的替代链 - 发生这种情况的可能性随着时间的推移而降低）。
接下来，我们以抽象的意义的形式介绍我们的框架，以尽可能保持通用性。
在第4节中，我们提出了一个满足要求并使用块DAG的协议。
到时候我们再来介绍该解决方案和挖坑协议的具体规定。
我们将使用我们的框架来阐明SPECTER避免比特币遭受的安全可扩展性折衷的意义。

## Transactions. 
## 交易
A transaction is typically denoted $tx$. $inputs (tx)$ is the set of transactions that must be accepted before $tx$ can be accepted; 
these are the transactions that have provided the money that is being spent in $tx$. 
Two different transactions $tx 1$ and $tx 2$ conﬂict if they share a common input, i.e., they double spend the same money; 
we then write $tx 2 ∈ conflict (tx 1 )$ (this is a symmetric relation).

交易通常表示为$tx$。$inputs (tx)$是$tx$可以被接受之前必须被接受的一组交易;
这些交易提供在$tx$上花费的资金。
两个不同的交易$tx 1$和$tx 2$如果共享一个共同的输入则它们是冲突的，即它们花费相同的资金;
我们以后表示为$tx 2 ∈ conflict (tx 1 )$（这是一个对称关系）。
**译注：** 对称是指 $tx 2 ∈ conflict (tx 1 )$ 同时也意味着 $tx 1 ∈ conflict (tx 2 )$

## Mining protocol. 
We denote by $N$ the set of nodes, aka miners. 
Miners maintain and extend the ledger, by adding transactions to it and propagating messages, according to the *mining protocol*. 
The propagation time of a message of size B KB to all nodes in the system is assumed to be under $D = D(B) $ seconds. 
For now, we regard the mining protocol as an abstract set of rules that miners must follow. 
We denote by honest the set of nodes that always follow the protocol’s instructions, and by malicious the complementary of this set.

## 挖坑协议 
我们用$N$来表示一组节点，又名矿工。 
根据*挖矿协议*，矿工通过添加交易并传播消息来维护和扩展账本。
大小为B KB的消息传播到系统中的所有节点的传播时间假定在 $D = D(B) $秒以内。 
目前，我们认为挖矿协议是矿工必须遵循的一套抽象规则。 
我们用诚实节点表示总是遵循协议指示的一组节点，并通过恶意节点表示非诚实节点。

In the family of protocols we focus on, miners possess computational power and perform proof-of-work (PoW). 
We denote by α the attacker’s relative computational power. 
Formally, it is the probability that the creator of the next PoW in the system belongs malicious; 
this is well deﬁned, as PoW creation is modeled as a memoryless process [13], [18], [17].

在我们关注的协议族中，矿工拥有计算能力并执行工作量证明（PoW）。 
我们用α表示攻击者的相对计算能力。 
形式上表示为，系统中下一个PoW的创建者有属于恶意节点的可能性;
这是很好的定义，因为PoW的创建是一个无记忆的过程[13]，[18]，[17]。
**译注：** 无记忆应该是指是纯随机的一个过程，每次随机都是独立的, 随机的结果不会记录

## Formation of ledger. 
The result of the mining protocol is an (abstract) public data structure G containing transactions (to be instantiated later, in our solution proposal, as a block DAG), aka the ledger. 
Nodes replicate the ledger locally. 
As they might hold slightly different views of the ledger (e.g., since blocks take time to propagate to all nodes), we denote by $G_t^v$ the state of the ledger as observed by node v at time t; 
we write $G_t$ when the local context is unimportant. 

## 账本的形成
挖矿协议的结果是一个包含交易的（抽象）公共数据结构G（稍后将在我们的解决方案提案中作为块DAG实例化），也就是账本。 
节点在本地有账本副本。 
由于它们可能保存着略有不同的账本状态（例如，因为区块需要花费时间传播到所有节点），所以我们用$G_t^v$表示时间t处由节点v观察到的账本的状态; 
当本地环境不重要时，我们写成$G_t$。 

## TxO protocol. 
Given a public ledger G, the TxO protocol extracts a consistent subset of transactions from G, denoted $TxO(G)$. 
Every transaction in this set must have its inputs in it as well, and cannot conﬂict with another transaction in the set.

## TxO协议
给定公共账本G，TxO协议从G中提取一致的交易子集，表示为$TxO(G)$。 
该集合中的每个交易都必须有其输入，并且不能与集合中的另一个交易冲突。

## Robust TxO protocol. 
Users of the system must get assurances regarding their payments. 
Basically, we want to guarantee that transactions will be accepted by all users, and that they will remain so forever. 
Given $G_t$, the RobustTxO protocol speciﬁes a subset of $G_t$, denoted $RobustTxO(G_t^v)$, which represents the set of accepted transactions that are guaranteed to remain so forever, up to an error probability of $ϵ$. RobustTxO takes as input $G_t^v$ (the local replicate of v ), and the values of D,λ,α, and $ϵ$. ^1 
A tx in (Robust) TxO is said to be (robustly) accepted. 

## 可靠 TxO 协议.
系统用户必须获得有关他们支付的保证。
基本上，我们希望保证交易将被所有用户接受，并且他们将永远保持。
给定$G_t$，RobustTxO协议指定$G_t$的一个子集，表示为$RobustTxO(G_t^v)$，它表示一系列被保证永久保留的已接受的交易，达到出错概率为$ϵ$。 
RobustTxO将输入$G_t^v$（v的本地复制）以及D，λ，α和$ϵ$的值作为输入。^1
(Robust) TxO 中的tx被认为是（可靠地）接受的。

## Desired properties. 
Thus, the following properties are essential for a cryptocurrency protocol:

## 所需的属性
因此，以下属性对于加密货币协议至关重要：

### Property 1 (Consistency). 
The accepted set is consistent: For any ledger G,

1) if $tx ∈ TxO(G)$ and $tx 2 ∈ inputs (tx)$ then $tx 2 ∈ TxO(G)$.

2) if $tx ∈ TxO(G)$ and $tx 2 ∈ conflict (tx)$ then $tx 2 ∉ TxO(G)$.

### 属性 1 (一致性). 
对于任何账本G，接受的集合满足如下条件则是一致的：

1) 如果$tx ∈ TxO(G)$且$tx 2 ∈ inputs (tx)$，则$tx 2 ∈ TxO(G)$。

2) 如果$tx ∈ TxO(G)$且$tx 2 ∈ conflict (tx)$，则$tx 2 ∉ TxO(G)$）。

### Property 2 (Safety). 
If a transaction is robustly accepted by some node, then w.h.p. it will be robustly accepted forever by all nodes, and the expected waiting time for this event is constant. 
Formally, $∀>0$ , $∀v ∈ N$ , if $tx ∈ RobustTxO(G_t^v , D, λ, α,ϵ)$, then w.p. of at least $1-ϵ$, there exists a time $τ ≥ t$ such that $∀u ∈ N$, $∀s ≥ τ : tx ∈ RobustTxO(G_s^u , D, λ, α,ϵ)$. 
If this event occurs, the expected value of $τ-t$ is constant.

### 属性 2 (安全性)
如果交易被某个节点可靠地接受，那么很大概率它将永远被所有节点永久接受，并且此事件的预期等待时间是恒定的。 
形式上表示为，$∀>0$ ，$∀v ∈ N$，如果$tx ∈ RobustTxO(G_t^v , D, λ, α,ϵ)$，则在概率至少为$1-ϵ$的情况下,存在时间$τ ≥ t$，使得$∀u ∈ N$, $∀s ≥ τ : tx ∈ RobustTxO(G_s^u , D, λ, α,ϵ)$。 
如果这个事件发生，$τ-t$的期望值是恒定的。

### Property 3 (Weak Liveness). 
If a transaction is published in the ledger, it is robustly accepted by any node after a short while, provided that its inputs are robustly accepted and that no conﬂicting transactions are published. 
Formally, let $v ∈ N$ , $tx ∈ G_t^v$ , and $ϵ>0$. 
Deﬁne by $ψ(t, D, λ, α,ϵ):= min\{s≥ t:tx ∈ RobustTxO(G_s^v,D,λ,α,ϵ)\}$ the waiting time for its  robust acceptance by v . 
Then, $\mathbb{E}[ψ-t|inputs(tx)⊆ TxO(G_ψ^v)∧ conflict(tx)∩ G_ψ^v=ϕ]$ is constant.

### 属性 3 (弱活性). 
如果一个交易已经在账本上发布，它可以在短时间内被任何节点可靠地接受，只要它的输入被可靠接受并且没有发布冲突的交易。 
形式上表示为，让 $v ∈ N$，$tx ∈ G_t^v$，并且$ϵ>0$。
定义为$ψ(t, D, λ, α,ϵ):= min\{s≥ t:tx ∈ RobustTxO(G_s^v,D,λ,α,ϵ)\}$，即被节点v可靠地接受所等待的时间。 
那么，$\mathbb{E}[ψ-t|inputs(tx)⊆ TxO(G_ψ^v)∧ conflict(tx)∩ G_ψ^v=ϕ]$是常数。

### Deﬁnition 1. 
The *security threshold* of a cryptocurrency protocol is deﬁned by the maximal α (attacker’s relative computational power) for which Properties 1-3 hold.

### 定义 1
加密货币协议的*安全阈值*定义为属性1-3所能承受的最大α（攻击者的相对计算能力）。

The expected values of $τ-t$ and $ψ-t$, as written in Properties 2 and 3, deﬁne the expected waiting time for conﬁrmation of transactions in the given protocol.

如属性2和3中所写的$τ-t$和$ψ-t$的期望值定义了在给定协议中确认交易的预期等待时间。

The “weakness” of the Liveness property corresponds to the fact that we do not guarantee (though it is still hard for an attacker to prevent) a resolution in case conﬂicting transactions were published soon one after the other. 
Contrast this to traditional consensus protocols, where all conﬂicts are required to be decided in ﬁnite time, a property usually referred to as Liveness. 
Observe, however, that an honest user of the system will never publish conﬂicting transactions, and will transfer money only after he robustly accepted the original funds (the inputs) himself; 
payments of honest users are thus guaranteed to meet the conditions formalized in Weak Liveness, and to be robustly accepted. 
On the other hand, an attacker trying to defraud must keep his attack secret before publishing the conﬂict, until the victim robustly accepts; 
but then the victim is guaranteed that w.h.p. his transaction will not be reversed. 
Therefore, these two properties together ensure that payments of honest users will be robustly accepted in constant expected time, and that they remain robustly accepted forever, w.h.p.

在活性属性的“弱”对应的结果是，我们不保证（虽然它仍然很难防止攻击者去这么做）能解决紧挨着的两笔冲突的交易。
与传统的共识协议进行对比，其中所有冲突都需要在有限时间内决定，通常称之为活性。
但是，考虑到该系统的一个诚实的用户不会发布冲突的交易，并只有在他亲自可靠地接受了来源资金（输入）后才会转账;
由此能保证诚实用户的支付符合弱活性中形式化的条件，并被可靠地接受。
另一方面，试图欺诈的攻击者在发布冲突交易之前必须保密，直到受害者可靠接受;
但是那时就可以保证受害者的交易极大地可能性不会被逆转。
因此，这两个属性共同确保诚实用户的支付将在恒定的预期时间内可靠接受，并将永远保持可靠接受.
**译注：** 这里的逻辑还是有点绕，举个例子说明，A 如果是攻击者是一个买家，B是受害者是一个卖家，A 不大可能连续发两笔双花的交易，因为这样很有可能B就能检测到双花交易而拒绝发货，更大的可能是A等B确认支付成功了并发货了，再发起一笔双花的交易。因为这个时候两笔双花交易间已经有足够长的时间了，不满足紧挨着的条件，SPECTRE是有办法保证活性的。

In this work we set out to design a protocol that can support a large throughput, and achieve fast conﬁrmation times, while maintaining a high security threshold.

在这项工作中，我们开始着手设计一个协议，该协议可以支持大吞吐量，并实现快速确认时间，同时保持高安全阈值。

^1
For the sake of clear exposition, we regard here these values as known. 
However, we emphasize that SPECTRE does not assume that nodes know or agree on the precise values of these parameters. 
See Section 3. 4
为了清楚说明，我们将这些值视为已知。 
但是，我们强调SPECTRE不会假设节点知道或同意这些参数的精确值。 
参见第3节
