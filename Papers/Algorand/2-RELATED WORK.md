
## 2 RELATED WORK
## 2   相关工作

**Proof-of-work.** Bitcoin [ 41 ], the predominant cryptocurrency, uses proof-of-work to ensure that everyone agrees on the set of approved transactions; this approach is often called “Nakamoto consensus”. 
Bitcoin must balance the length of time to compute a new block with the possibility of wasted work [ 41 ], and sets parameters to generate a new block every 10 minutes on average.
 Nonetheless, due to the possibility of forks, it is widely suggested that users wait for the blockchain to grow by at least six blocks before considering their transaction to be confirmed [7].
 This means transactions in Bitcoin take on the order of an hour to be confirmed.
 Many follow-on cryptocurrencies adoptBitcoin’s proof-of-work approach and inherit its limitations.
The possibility of forks also makes it difficult for new users to bootstrap securely: an adversary that isolates the user’s network can convince the user to use a particular fork of the blockchain [28].

**工作量证明.** 比特币[41]是最主要的加密货币，它使用工作量证明来确保每个人都同意一组已确认的交易。
这种方法通常被称为“中本聪共识”。
比特币必须在计算新块的时间和可能浪费的工作量之间平衡[41]，并设置参数以确保平均每10分钟生成一个新块。
尽管如此，由于分叉的可能性，人们普遍认为用户需要等待区块链增长至少六个区块之后才考虑确认他们的交易[7]。
这意味着比特币交易需要一个小时才能确认。
许多后续加密货币采用比特币的工作量证明方法，也因此延续了其限制。
分叉的可能性也使得新用户难以安全地加入网络：攻击者可以隔离用户的网络， 然后说服用户使用区块链的特定分支[28]。


By relying on Byzantine agreement, Algorand eliminates the possibility of forks, and avoids the need to reason about mining strategies [ 8 , 25 , 46 ].
 As a result, transactions are confirmed on the order of a minute.
 To make the Byzantine agreement robust to Sybil attacks, Algorand associates weights with users according to the money they hold.
 Other techniques have been proposed in the past to resist Sybil attacks in Byzantine-agreement-based cryptocurrencies, including having participants submit security deposits and punishing those who deviate from the protocol [13].
 
依靠拜占庭协议，Algorand消除了分叉的可能性，并避免了制定挖矿策略[8,25,46]。
结果，交易可以在一分钟的规模内确认。
为了使拜占庭协议能抵抗女巫攻击，Algorand根据用户所持有的钱将权重与用户关联起来。
已经有其他基于拜占庭协议加密货币的技术提出来抵抗女巫攻击，包括让参与者提交安全保证金并惩罚那些违法协议[13]的人。

**Byzantine consensus.** Byzantine agreement protocols have been used to replicate a service across a small group of servers, such as in PBFT [ 15 ].
 Follow-on work has shown how to make Byzantine fault tolerance perform well and scale to dozens of servers [ 1 , 17 , 33 ].
 One downside of Byzantine fault tolerance protocols used in this setting is that they require a fixed set of servers to be determined ahead of time;
 allowing anyone to join the set of servers would open up the protocols to Sybil attacks.
 These protocols also do not scaleto the large number of users targeted by Algorand.
BA⋆is a Byzantine consensus protocol that does not rely on a fixed set of servers, which avoids the possibility of targeted attacks on well-known servers.
 By weighing users according to their currency balance,BA⋆allows users to join the cryptocurrency without risking Sybil attacks, as long as the fraction of the money held by honest users is at least a constant greater than 2/3.
BA⋆’s design also allows it to scale to many users(e.g., 500,000 shown in our evaluation) using VRFs to fairly select a random committee.

**拜占庭共识.** 拜占庭共识协议被用来在一小组服务器中同步数据，如PBFT [ 15 ]。
接下来的工作展示了如何使拜占庭容错性能良好，并扩展到更多台服务器上[ 1, 17, 33 ]。
拜占庭容错协议设定的一个缺点是，它们需要提前确定一套固定的服务器；因此若允许任何人加入服务器组将可能遭受女巫攻击。
这些协议也不能扩展到满足Algorand用户数量的要求。
BA* 是一个拜占庭共识的协议，不依赖于固定的一组服务器，避免了针对已知服务器攻击的可能性。
根据货币余额确定用户权重，BA* 允许用户加入网络而不用担心被女巫攻击，前提是诚实的用户总余额比例至少为2/3。
BA* 的设计也可以扩展到许多用户（例如，在我们的评估中是500000），用以公平地选择随机委员会。

Most Byzantine consensus protocols require more than 2 / 3 of servers to be honest, and Algorand’s BA⋆inherits this limitation (in the form of 2 / 3 of the money being held by honest users).
 BFT2F [ 35 ] shows that it is possible to achieve “fork∗-consensus” with just over half of the servers being honest, but fork∗-consensus would allow an adversary to double-spend on the two forked blockchains, which Algorand avoids.
 
 大多数的拜占庭式共识协议需要超过2/3的服务器是诚实的，Algorand BA* 通过让诚实用户持有超过2/3货币的形式延续了这个限制。
 BFT2F [ 35 ]表明，只需要服务器超过1/2是诚实也有可能实现“fork∗-consensus”，但"fork∗-consensus"允许攻击者在两个分叉的区块链上双花，而Algorand避免了这个问题。
 
Honey Badger [ 39 ] demonstrated how Byzantine fault tolerance can be used to build a cryptocurrency.
 Specifically,Honey Badger designates a set of servers to be in charge of reaching consensus on the set of approved transactions.
This allows Honey Badger to reach consensus within 5 minutes and achieve a throughput of 200 KBytes/sec of data appended to the ledger using 10 MByte blocks and 104 participating servers.
 One downside of this design is that the cryptocurrency is no longer decentralized; there are a fixed set of servers chosen when the system is first configured.
Fixed servers are also problematic in terms of targeted attacks that either compromise the servers or disconnect them from the network.
 Algorand achieves better performance(confirming transactions in about a minute, reaching similar throughput) without having to choose a fixed set of servers ahead of time.
 
Honey Badger[ 39 ]展示了拜占庭容错可以用来创建一个加密货币。
具体而言，Honey Badger指定了一组服务器，负责在已确认的交易集上达成共识。
在使用10M区块和104个参与服务器的网络下，Honey Badger以200k/s的吞吐量追加数据到账本，可以在5分钟内达到共识。
这种设计的一个缺点是，加密货币不再是去中心的；在系统首次配置时要选定一组固定的服务器。
固定服务器容易遭受信息泄漏和拒绝服务攻击。 
**译注：** 这里用的是compromise，compromise和hack是有区别的，参见https://us.battle.net/forums/en/d3/topic/5149544320
Algorand可以达到更好的性能（在大约一分钟内确认交易，并达到近似的吞吐量），且无需提前选择一组固定的服务器。
 
Bitcoin-NG [ 26 ] suggests using the Nakamoto consensus to elect a leader, and then have that leader publish blocks of transactions, resulting in an order of magnitude of improvement in latency of confirming transactions over Bitcoin.
Hybrid consensus [ 30 , 32 , 42 ] refines the approach of using the Nakamoto consensus to periodically select a group of participants (e.g., every day), and runs a Byzantine agreement between selected participants to confirm transactions until new servers are selected.
 This allows improving performance over standard Nakamoto consensus (e.g., Bitcoin); for example, ByzCoin [ 32 ] provides a latency of about 35 seconds and a throughput of 230 KBytes/sec of data appended to the ledger with an 8 MByte block size and 1000 participants in the Byzantine agreement.
 Although Hybrid consensus makes the set of Byzantine servers dynamic, it opens up the possibility of forks, due to the use of proof-of-work consensus to agree on the set of servers; this problem cannot arise in Algorand.
 
Bitcoin-NG[ 26 ]建议使用中本聪共识来选出一个领导者，然后由领导者发布交易区块，从而在确认交易延迟方面比特币提升了一个数量级。
混合共识（Hybrid consensus）[ 30, 32, 42 ]改进了中本聪共识，定期（例如，每天）选择一组参与者，并在选定参与者中运行一个拜占庭共识来确认交易，直到到新的服务器被选定。
这样可以提高标准中本聪共识性能（例如，比特币）；例如，ByzCoin[ 32 ]的延迟为35秒左右，在8M区块的大小和1000参与者的网络下可以达到230K字节/秒的账本写入的吞吐量。
虽然混合共识的选出拜占庭服务器组是动态的，但它仍然可能分叉，因为该组服务器使用pow共识来达成共识；这个问题不会出现在Algorand里。
 
 
Pass and Shi’s paper [ 42 ] acknowledges that the Hybrid consensus design is secure only with respect to a “mildly adaptive” adversary that cannot compromise the selected servers within a day (the participant selection interval), and explicitly calls out the open problem of handling fully adaptive adversaries.
 Algorand’s BA⋆ explicitly addresses this open problem by immediately replacing any chosen committee members.
 As a result, Algorand is not susceptible to either targeted compromises or targeted DoS attacks.
 
 Pass and Shi 的论文指出， 混合共识的设计只针对那种温和的攻击者是安全的，这种攻击者在一天之内（参与者选举间隔）无法使服务器瘫痪，并明确提出了处理完全自适应的攻击者时的开放性问题。
 Algorand的BA* 算法针对这个问题提出了解决办法，即立刻更换被选中的委员会成员。
 因此，Algorand不会受到针对性的信息泄漏或拒绝服务攻击的影响。
 
Stellar [ 36 ] takes an alternative approach to using Byzantine consensus in a cryptocurrency, where each user can trust quorums of other users, forming a trust hierarchy.
 Consistency is ensured as long as all transactions share at least one transitively trusted quorum of users, and sufficiently many of these users are honest.
 Algorand avoids this assumption, which means that users do not have to make complex trust decisions when configuring their client software.
 
 恒星币[ 36 ]以另一种方法使用拜占庭式的共识，即每个用户可以信任其他用户组成的仲裁团（quorum），形成信任层级。
 只要所有交易共享至少一个可信赖的仲裁团的全部用户，并且这些用户有足够多的人都是诚实的，共识就能够达成。
 Algorand并不需要这个前提，这意味着用户在配置客户端软件时不必做出复杂的信任决策。
 
 
**Proof of stake.** Algorand assigns weights to users proportionally to the monetary value they have in the system, inspired by proof-of-stake approaches, suggested as an alternative or enhancement to proof-of-work [ 3 , 10 ].
 There is a key difference, however, between Algorand using monetary value as weights and many proof-of-stake cryptocurrencies.
In many proof-of-stake cryptocurrencies, a malicious leader(who assembles a new block) can create a fork in the network, but if caught (e.g., since two versions of the new block are signed with his key), the leader loses his money.
 The weights in Algorand, however, are only to ensure that the attacker cannot amplify his power by using pseudonyms; as long as the attacker controls less than 1/3 of the monetary value, Algorand can guarantee that the probability for forks is negligible.
 Algorand may be extended to “detect and punish” malicious users, but this is not required to prevent forks or double spending.
 
** 股权证明.**  股权证明，是一种建议替代或改进POW的共识算法，Algorand从中得到启发，根据用户在系统中的货币价值的比例分配权重。
但是Algorand和许多股权证明加密货币在使用货币价值量作为权重的规则上有着很关键的区别。
在许多其他股权证明的加密货币中，一个恶意的见证人在生产出一个区块后可以在网络中创建分叉，但是如果被抓住（例如， 两个版本的新区块都被他签名），这个见证人将丢失他的钱。
在Algorand中，权重只是确保攻击者无法通过使用虚假账号来增强他的能力；只要攻击者控制的货币价值少于1/3，Algorand能够保证分叉的可能性是微乎其微的。
Algorand也可以支持“检测和惩罚“恶意用户，但这对于防止分叉或双花来说不是必要的。
  
 
Proof-of-stake avoids the computational overhead of proof-of-work and therefore allows reducing transaction confirmation time.
 However, realizing proof-of-stake in practice is challenging [ 4 ].
 Since no work is involved in generating blocks, a malicious leader can announce one block, and then present some other block to isolated users.
 Attackers may also split their credits among several “users”, who might get selected as leaders, to minimize the penalty when a bad leader is caught.
 Therefore some proof-of-stake cryptocurrencies require a master key to periodically sign the correct branch of the ledger in order to mitigate forks [ 31 ].
 This raises significant trust concerns regarding the currency, and has also caused accidental forks in the past [ 43 ].
 Algorand answers this challenge by avoiding forks, even if the leader turns out to be malicious.
 
 股权证明避免了工作量证明的计算开销，因此可以减少交易确认时间。
 然而，在实践中也需要认识到股权证明是有挑战性的[ 4 ]。
 由于没有涉及生成块的工作，恶意的领导者可以发布一个块，然后向被隔离的用户呈现其他的块。
 攻击者也可以把他们的存款分成几个“用户”，他们可能会被选为领导者，从而当一个坏领导被抓住时，将惩罚降至最低。
 因此，一些股权证明的加密货币需要一个主key在账本的正确分支上周期性地签名，来减少分叉。
 这引发了对货币的重大信任问题，并在过去[ 43 ]中也引起了意外的分叉。
 Algorand通过避免分叉解决了这一挑战，即使领导者是恶意的。
 
Ouroboros [ 30 ] is a recent proposal for realizing proof-of-stake.
 For security, Ouroboros assumes that honest users can communicate within some bounded delay (i.e., a strongly synchronous network).
 Furthermore, it selects some users to participate in a joint-coin-flipping protocol and assumes that most of them are incorruptible by the adversary for a significant epoch (such as a day).
 In contrast Algorand assumes that the adversary may temporarily fully control the network and immediately corrupt users in targeted attacks.
 **Ouroboros[ 30 ]** 是一个最近的实现股权证明的提议。
 为了安全，Ouroboros假定诚实的用户能够在一些有限的延迟里进行沟通（即，一个强同步网络）。
 此外，它还选择了一些用户参与一个联合掷币（joint-coin-flipping）协议，并假设他们中的大部分人在一个重要的时代（epoch，比如一天）不被攻击者攻击。
 相反，Algorand认为攻击者可能临时完全控制网络并立即对用户发动针对性攻击。
 
**Trees and DAGs instead of chains.** GHOST [ 47 ], SPECTRE [ 48 ], and Meshcash [ 5 ] are recent proposals for increasing Bitcoin’s throughput by replacing the underlying chain structured ledger with a tree or directed acyclic graph (DAG) structures, and resolving conflicts in the forks of these data structures.
 These protocols rely on the Nakamoto consensus using proof-of-work.
 By carefully designing the selection rule between branches of the trees/DAGs, they are able to substantially increase the throughput.
 In contrast, Algorand is focused on eliminating forks; in future work, it may be interesting to explore whether tree or DAG structures can similarly increase Algorand’s throughput.
 
 **使用树或者有向无环图代替链** GHOST [ 47 ], SPECTRE [ 48 ], and Meshcash [ 5 ] 是最近用来提升比特币吞吐量的一些提议，它们通过使用树或者有向无环图来代替底层区块链的账本结构，并且解决区块链分叉的冲突问题。
 这些协议依赖于使用工作量证明的中本聪共识。
 通过精心设计的树/图分支之间的选择规则，能够大幅提高吞吐量。
 相比之下，Algorand的重点是消除分叉；在今后的工作中，研究树或DAG结构是否可以采用类似的方法增加Algorand的吞吐量可能会很有趣。
 
 翻译：伍歌歌wytiger
校订： DAG fans
