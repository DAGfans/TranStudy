## 4 OVERVIEW

Algorand requires each user to have a public key. Algorand maintains a log of transactions, called a blockchain. Each transaction is a payment signed by one user’s public key transferring money to another user’s public key. Algorand grows the blockchain in asynchronous rounds, similar to Bitcoin. In every round, a new block, containing a set of transactions and a pointer to the previous block, is appended to the blockchain. In the rest of this paper, we refer to Algorand software running on a user’s computer as that user.

Algorand要求每个用户都拥有一个公钥。 Algorand所维护的交易记录，称为区块链。每项交易都是由一个经用户签名的公钥支付到另一个用户的公钥上。Algorand以异步轮增长区块链，与比特币类似。在每一轮中，一个新区块将包含一组交易包和指向前序区块的指针。在本文的其余部分，我们将在用户计算机上运行的Algorand软件称为该用户。
译注：这里的异步指通一个区块里面的交易不是严格按时间排序的


Algorand users communicate through a gossip protocol. The gossip protocol is used by users to submit new transactions. Each user collects a block of pending transactions that they hear about, in case they are chosen to propose the next block, as shown in Figure 1. Algorand uses BA⋆ to reach consensus on one of these pending blocks.

Algorand用户通过流言（gossip）协议进行通信。流言协议被用户用来提交新的交易。如图1所示，每个用户收集一个他们发现的待处理交易块，以供他们选择提议下一个块。Algorand使用BA⋆对这些待处理的区块之一达成共识。

figure 1



Figure 1: An overview of transaction flow in Algorand. Algorand中的交易流概览



BA⋆ executes in steps, communicates over the same gossip protocol, and produces a new agreed-upon block. BA⋆ can produce two kinds of consensus: final consensus and tentative consensus. If one user reaches final consensus, this means that any other user that reaches final or tentative consensus in the same round must agree on the same block value (regardless of whether the strong synchrony assumption held). This ensures Algorand’s safety, since this means that all future transactions will be chained to this final block (and, transitively, to its predecessors). Thus, Algorand confirms a transaction when the transaction’s block (or any successor block) reaches final consensus. On the other hand, tentative consensus means that other users may have reached tentative consensus on a different block (as long as no user reached final consensus). A user will confirm a transaction from a tentative block only if and when a successor block reaches final consensus.

BA⋆是逐步执行的，它通过同一个gossip协议层进行通信，并产生一个新的共识区块。BA⋆可以产生两种共识：最终共识和暂定共识。如果一个用户达成最终共识，这意味着在同一轮中达成最终或暂定共识的任何其他用户必须就相同的区块价值达成一致（不管强同步假设是否成立）。这确保了Algorand的安全性，因为这意味着所有未来的交易都将被链接到这个最后的区块（并且传递到其前置任务）。因此，当交易的区块（或任何后继区块）达到最终共识时，Algorand才确认交易。另一方面，暂定共识意味着其他用户可能在不同的区块达成暂定共识（只要没有用户达成最终共识）。 只有当后继区块达成最终共识时，用户才会确认来自暂定区块的交易。
译注：这里的强同步指的是诚实节点可以在有限的时间内能够沟通


BA⋆ produces tentative consensus in two cases. First, if the network is strongly synchronous, an adversary may, with small probability, be able to cause BA⋆ to reach tentative consensus on a block. In this case, BA⋆ will not reach consensus on two different blocks, but is simply unable to confirm that the network was strongly synchronous. Algorand will eventually (in a few rounds) reach final consensus on a successor block, with overwhelming probability, and thus confirm these earlier transactions.

BA⋆在两种情况下中产生暂定共识。首先，如果网络强同步的，攻击者可能以较小的概率使BA⋆在一个区块上达成暂定共识。在这种情况下，BA⋆不会在两个不同的区块上达成共识，但也无法简单的确定网络是否是强同步的。 Algorand最终（在几轮中）将以最大的可能性在后续区块上达到最终的共识，进而来确认先前的交易。



The second case is that the network was only weakly synchronous (i.e., it was entirely controlled by the adversary, with an upper bound on how long the adversary can keep control). In this case, BA⋆ can reach tentative consensus on two different blocks, forming multiple forks. This can in turn prevent BA⋆ from reaching consensus again, because the users are split into different groups that disagree about previous blocks. To recover liveness, Algorand periodically invokes BA⋆to reach consensus on which fork should be used going forward. Once the network regains strong synchrony, this will allow Algorand to choose one of the forks, and then reach final consensus on a subsequent block on that fork.

第二种情况是网络只是弱同步的（即它完全由攻击者控制，但攻击者只能够在有限的时间里保持对网络的控制）。在这种情况下，BA⋆可以在两个不同的区块上达成暂定共识，形成多个分叉。 这反而可以阻止BA⋆再次达成共识，因为用户按照对前序区块有分歧的形式被分成不同的组。为了恢复活性（Liveness），Algorand定期调用BA⋆来选择应该使用哪个分叉来向前达成共识。一旦网络恢复强同步性，将允许Algorand选择其中一个分叉，然后就在该分叉的后续区块上达成最终共识。



We now describe how Algorand’s components fit together.

现在，我们介绍Algorand的组件如何组合在一起。



**Gossip protocol.** Algorand implements a gossip network (similar to Bitcoin) where each user selects a small random set of peers to gossip messages to. To ensure messages cannot be forged, every message is signed by the private key of its original sender; other users check that the signature is valid before relaying it. To avoid forwarding loops, users do not relay the same message twice. Algorand implements gossip over TCP and weighs peer selection based on how much money they have, so as to mitigate pollution attacks.

**流言协议.**  Algorand实现了流言（Gossip）网络（类似于比特币），在这里，每个用户都随机选择很小一部分节点来将消息传播出去。为确保消息不被伪造，每个消息都由其原始发件人用私钥签名; 其他用户在转发之前要检查签名是否有效。为避免转发循环，用户不会转发相同的消息两次。 Algorand通过TCP实现了流言协议，并基于他们拥有的钱来权衡节点的选择，从而减轻污染攻击。
**译注：** 这里的污染典型的就是女巫攻击

**Block proposal (§6).**  All Algorand users execute cryptographic sortition to determine if they are selected to propose a block in a given round. We describe sortition in §5, but at a high level, sortition ensures that a small fraction of users are selected at random, weighed by their account balance, and provides each selected user with a priority, which can be compared between users, and a proof of the chosen user’s priority. Since sortition is random, there may be multiple users selected to propose a block, and the priority determines which block everyone should adopt. Selected users distribute their block of pending transactions through the gossip protocol, together with their priority and proof. To ensure that users converge on one block with high probability, block proposals are prioritized based on the proposing user’s priority, and users wait for a certain amount of time to receive the block.

**区块提议（§6）.** 所有Algorand用户执行密码学的抽签以确定他们是否被选择在给定的轮次中提议一个块。 我们在§5中会详细介绍抽签，但是在较高层次上可以理解为，根据用户帐户余额的权重，随机抽选出一小部分用户，并为每个选定的用户提供可以在用户与用户之间进行比较的优先级以及所选用户的优先权的证明。由于抽签是随机的，因此可能有多个用户被选择来提议一个区块，优先级将决定每个人应采用哪个区块。选定的用户通过流言协议分发他们待处理的交易区块以及他们的优先权和证明。为了确保用户以较高的概率收敛到一个区块，区块提议的优先级基于提议用户的优先级，用户等待一定的时间后来接收区块。



**Agreement using BA⋆ (§7).** Block proposal does not guarantee that all users received the same block, and Algorand does not rely on the block proposal protocol for safety. To reach consensus on a single block, Algorand uses BA⋆. Each user initializes BA⋆ with the highest-priority block that they received. BA⋆ executes in repeated steps, illustrated in Figure 2. Each step begins with sortition (§5), where all users check whether they have been selected as committee members in that step. Committee members then broadcast a message which includes their proof of selection. These steps repeat until, in some step of BA⋆, enough users in the committee reach consensus. (Steps are not synchronized across users; each user checks for selection as soon as he observes the previous step had ended. ) As discussed earlier, an important feature of BA⋆ is that committee members do not keep private state except their private keys, and so can be replaced after every step, to mitigate targeted attacks on them.

**使用BA⋆协议（§7）达成共识.** 区块提议并不保证所有用户都收到相同的区块，并且Algorand不依赖区块提议协议以确保安全。为了达成一个单独区块的共识，Algorand使用BA⋆。每个用户使用他们收到的优先级最高的区块来初始化BA⋆。BA⋆以重复的步骤执行，如图2所示。每个步骤都从抽签（§5）开始，所有用户都会检查他们是否在该步骤中被选为委员会成员。委员会成员随后广播了一条包含他们被选择的证明的信息。重复这些步骤直到在BA⋆的某个步骤中，委员会中有足够的用户达成共识。 （步骤在用户之间不同步；每当用户观察到前一步骤结束时，每个用户都会检查是否被选择。）如前所述，BA⋆的一个重要特征是委员会成员除私钥以外不保留私有状态，因此可以在每一步后更换，以减轻对它们的有针对性的攻击。



Figure 2: An overview of one step of BA⋆. To simplify the figure, each user is shown twice: once at the top of the diagram and once at the bottom. Each arrow color indicates a message from a particular user.

图2：BA⋆一个步骤的概述。为了简化该图，每个用户都只显示两次：一次位于图顶部，一次位于底部。 每个箭头的颜色表示来自特定用户的消息。



**Efficiency.**  When the network is strongly synchronous, BA⋆ guarantees that if all honest users start with the same initial block (i.e., the highest priority block proposer was honest), then BA⋆ establishes final consensus over that block and terminates precisely in 4 interactive steps between users. Under the same network conditions, and in the worst case of a particularly lucky adversary, all honest users reach consensus on the next block within expected 13 steps, as analyzed in Appendix C.

**效率.** 当网络是强同步时，BA⋆保证如果所有诚实用户都从相同的初始区块开始（也就是最高优先级区块提议者是诚实的），那么BA⋆会在该块上建立最终共识并且在会用户之间恰好交互四次后终结。在相同的网络条件下，并且在最坏的情况下，即便出现了特别幸运的攻击者，也能保证所有诚实的用户在预期的13个步骤内达成对下一个块的共识，具体参看附录C中的分析。
