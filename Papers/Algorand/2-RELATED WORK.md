## 2 RELATED WORK
相关工作
**Proof-of-work.** Bitcoin [ 41 ], the predominant cryptocurrency, uses proof-of-work to ensure that everyone agrees on the set of approved transactions; this approach is often called “Nakamoto consensus”. 
Bitcoin must balance the length of time to compute a new block with the possibility of wasted work [ 41 ], and sets parameters to generate a new block every 10 minutes on average.
 Nonetheless, due to the possibility of forks, it is widely suggested that users wait for the blockchain to grow by at least six blocks before considering their transaction to be confirmed [7].
 This means transactions in Bitcoin take on the order of an hour to be confirmed.
 Many follow-on cryptocurrencies adoptBitcoin’s proof-of-work approach and inherit its limitations.
The possibility of forks also makes it difficult for new users to bootstrap securely: an adversary that isolates the user’s network can convince the user to use a particular fork of the blockchain [28].
工作量证明。比特币，主要的加密货币，使用工作量证明来确保每个人都认可区块的交易集合（简单说就是使用pow达成共识），这种共识机制经常被叫做“中本聪共识”。
比特币必须平衡计算新块的时间长度和浪费工作的可能性[ 41 ]，并设置参数以平均每10分钟生成一个新块。尽管如此，因为有分叉的可能性，人们普遍认为用户等待区块链增长至少六块才考虑他们的交易被确认[ 7 ]。这意味着比特币的交易需要一个小时的时间才能得到确认。许多后续的加密货币采用了比特币的pow同时也继承了他的局限性。分叉的可能性也使得引导新用户安全性变得困难：攻击者通过隔离用户网络，使用户使用一个特别的区块链分叉。


By relying on Byzantine agreement, Algorand eliminates the possibility of forks, and avoids the need to reason about mining strategies [ 8 , 25 , 46 ].
 As a result, transactions are confirmed on the order of a minute.
 To make the Byzantine agreement robust to Sybil attacks, Algorand associates weights with users according to the money they hold.
 Other techniques have been proposed in the past to resist Sybil attacks in Byzantine-agreement-based cryptocurrencies, including having participants submit security deposits and punishing those who deviate from the protocol [13].
依靠拜占庭协议，algorand消除了分叉的可能性，避免了挖矿的需要。因此，交易按一分钟就可以完成。为了使拜占庭协议面对对女巫攻击任然健壮，algorand将他们持有的资金与权重相关联。基于拜占庭协议的货币中，其他的技术在过去已经被提出以抵抗女巫攻击，包括参加者提交保证金和惩罚那些作恶者[ 13 ]。

**Byzantine consensus.** Byzantine agreement protocols have been used to replicate a service across a small group of servers, such as in PBFT [ 15 ].
 Follow-on work has shown how to make Byzantine fault tolerance perform well and scale to dozens of servers [ 1 , 17 , 33 ].
 One downside of Byzantine fault tolerance protocols used in this setting is that they require a fixed set of servers to be determined ahead of time;
 allowing anyone to join the set of servers would open up the protocols to Sybil attacks.
 These protocols also do not scaleto the large number of users targeted by Algorand.
BA⋆is a Byzantine consensus protocol that does not rely on a fixed set of servers, which avoids the possibility of targeted attacks on well-known servers.
 By weighing users according to their currency balance,BA⋆allows users to join the cryptocurrency without risking Sybil attacks, as long as the fraction of the money held by honest users is at least a constant greater than 2/3.
BA⋆’s design also allows it to scale to many users(e.g., 500,000 shown in our evaluation) using VRFs to fairly select a random committee.
拜占庭式共识。拜占庭协议被重写改进，通过一小组服务器，如PBFT [ 15 ]。接下来的工作展示了如何使拜占庭容错性能良好，并扩展到更多台服务器上[ 1, 17, 33 ]。拜占庭容错协议使用此设置的一个缺点是，它们需要提前确定一套固定的服务器；允许任何人加入服务器集合将使得对女巫攻击开放。这些协议还不能扩展到满足algorand用户数量。BA⋆是一个拜占庭共识的协议，不依赖于固定的一组服务器，避免了针对已知服务器攻击的可能性。根据他们的货币余额的权衡用户权重，BA⋆允许用户不用冒Sybil攻击风险加入货币网络，只要诚实的用户余额比例至少是大于2/3的常数。BA⋆的设计也可以扩展到许多用户（例如，在我们的评估中是500000），用以公平地选择随机委员会。



Most Byzantine consensus protocols require more than2 / 3 of servers to be honest, and Algorand’s BA⋆inherits this limitation (in the form of 2 / 3 of the money being held by honest users).
 BFT2F [ 35 ] shows that it is possible to achieve “fork∗-consensus” with just over half of the servers being honest, but fork∗-consensus would allow an adversary to double-spend on the two forked blockchains, which Algorand avoids.
 
Honey Badger [ 39 ] demonstrated how Byzantine fault tolerance can be used to build a cryptocurrency.
 Specifically,Honey Badger designates a set of servers to be in charge of reaching consensus on the set of approved transactions.
This allows Honey Badger to reach consensus within 5 minutes and achieve a throughput of 200 KBytes/sec of data appended to the ledger using 10 MByte blocks and 104 participating servers.
 One downside of this design is that the cryptocurrency is no longer decentralized; there are a fixed set of servers chosen when the system is first configured.
Fixed servers are also problematic in terms of targeted attacks that either compromise the servers or disconnect them from the network.
 Algorand achieves better performance(confirming transactions in about a minute, reaching similar throughput) without having to choose a fixed set of servers ahead of time.
 
Bitcoin-NG [ 26 ] suggests using the Nakamoto consensus to elect a leader, and then have that leader publish blocks of transactions, resulting in an order of magnitude of improvement in latency of confirming transactions over Bitcoin.
Hybrid consensus [ 30 , 32 , 42 ] refines the approach of using the Nakamoto consensus to periodically select a group of participants (e.g., every day), and runs a Byzantine agreement between selected participants to confirm transactions until new servers are selected.
 This allows improving performance over standard Nakamoto consensus (e.g., Bitcoin); for example, ByzCoin [ 32 ] provides a latency of about 35 seconds and a throughput of 230 KBytes/sec of data appended to the ledger with an 8 MByte block size and 1000 participants in the Byzantine agreement.
 Although Hybrid consensus makes the set of Byzantine servers dynamic, it opens up the possibility of forks, due to the use of proof-of-work consensus to agree on the set of servers; this problem cannot arise in Algorand.
 
Pass and Shi’s paper [ 42 ] acknowledges that the Hybrid consensus design is secure only with respect to a “mildly adaptive” adversary that cannot compromise the selected servers within a day (the participant selection interval), and explicitly calls out the open problem of handling fully adaptive adversaries.
 Algorand’sBA⋆explicitly addresses thisopen problem by immediately replacing any chosen committee members.
 As a result, Algorand is not susceptible to either targeted compromises or targeted DoS attacks.
 
Stellar [ 36 ] takes an alternative approach to using Byzantine consensus in a cryptocurrency, where each user can trust quorums of other users, forming a trust hierarchy.
 Consistency is ensured as long as all transactions share at least one transitively trusted quorum of users, and sufficiently many of these users are honest.
 Algorand avoids this assumption, which means that users do not have to make complex trust decisions when configuring their client software.
 
**Proof of stake.** Algorand assigns weights to users proportionally to the monetary value they have in the system, inspired by proof-of-stake approaches, suggested as an alternative or enhancement to proof-of-work [ 3 , 10 ].
 There is a key difference, however, between Algorand using monetary value as weights and many proof-of-stake cryptocurrencies.
In many proof-of-stake cryptocurrencies, a malicious leader(who assembles a new block)can create a fork in the network, but if caught (e.g., since two versions of the new block are signed with his key), the leader loses his money.
 The weights in Algorand, however, are only to ensure that the attacker cannot amplify his power by using pseudonyms; as long as the attacker controls less than 1/3 of the monetary value, Algorand can guarantee that the probability for forks is negligible.
 Algorand may be extended to “detect and punish” malicious users, but this is not required to prevent forks or double spending.
 
Proof-of-stake avoids the computational overhead of proof-of-work and therefore allows reducing transaction confirmation time.
 However, realizing proof-of-stake in practice is challenging [ 4 ].
 Since no work is involved in generating blocks, a malicious leader can announce one block, and then present some other block to isolated users.
 Attackers may also split their credits among several “users”, who might get selected as leaders, to minimize the penalty when a bad leader is caught.
 Therefore some proof-of-stake cryptocurrencies require a master key to periodically sign the correct branch of the ledger in order to mitigate forks [ 31 ].
 This raises significant trust concerns regarding the currency, and has also caused accidental forks in the past [ 43 ].
 Algorand answers this challenge by avoiding forks, even if the leader turns out to be malicious.
 
Ouroboros [ 30 ] is a recent proposal for realizing proof-ofstake.
 For security, Ouroboros assumes that honest users can communicate within some bounded delay (i.e., a strongly synchronous network).
 Furthermore, it selects some users to participate in a joint-coin-flipping protocol and assumes that most of them are incorruptible by the adversary fora significant epoch (such as a day).
 In contrast Algorand assumes that the adversary may temporarily fully control the network and immediately corrupt users in targeted attacks.
 
**Trees and DAGs instead of chains.** GHOST [ 47 ], SPECTRE [ 48 ], and Meshcash [ 5 ] are recent proposals for increasing Bitcoin’s throughput by replacing the underlying chain structured ledger with a tree or directed acyclic graph (DAG)structures, and resolving conflicts in the forks of these data structures.
 These protocols rely on the Nakamoto consensus using proof-of-work.
 By carefully designing the selection rule between branches of the trees/DAGs, they are able to substantially increase the throughput.
 In contrast, Algorand is focused on eliminating forks; in future work, it may be interesting to explore whether tree or DAG structures can similarly increase Algorand’s throughput.
