## 7 BA⋆

The execution of BA⋆ consists of two phases. In the first phase, BA⋆ reduces the problem of agreeing on a block to agreement on one of two options. In the second phase, BA⋆ reaches agreement on one of these options: either agreeing on a proposed block, or agreeing on an empty block.

BA⋆的执行由两个阶段组成。 在第一阶段，BA⋆ 把共识一个区块的问题降解到共识两个选择中的一个。在第二阶段，BA⋆就这些选择之一达成一致：要么同意提议的区块，要么同意空的区块

Each phase consists of several interactive steps; the first phase always takes two steps, and the second phase takes two steps if the highest-priority block proposer was honest (sent the same block to all users), and as we show in our analysis an expected 11 steps in the worst case of a malicious highest-priority proposer colluding with a large fraction of committee participants at every step.

每个阶段由一些相互作用的步骤组成，第一阶段总是由2步组成，第二阶段如果最高优先级的出块者是诚实的那么花费2步，并在我们的分析中如果攻击者是出块者并和大部分委员会成员合谋的最坏情况下花费11步。

In each step, every committee member casts a vote for some value, and all users count the votes. Users that receive more than a threshold of votes for some value will vote for that value in the next step (if selected as a committee member). If the users do not receive enough votes for any value, they time out, and their choice of vote for the next step is determined by the step number.

在每个步骤中，每个委员会成员都会为一些value投票，所有用户都会对选票进行计数。 获得超过某个value选票的阈值的用户将在下一步（如果选为委员会成员）中投票选择该值。 如果用户没有获得足够的任何value的投票，他们超时，并对下一步投票的选择是由步骤号确定。

In the common case, when the network is strongly synchronous and the highest-priority block proposer was honest, BA⋆ reaches final consensus by using its final step to confirm that there cannot be any other agreed-upon block in the same round. Otherwise, BA⋆ may declare tentative consensus if it cannot confirm the absence of other blocks due to possible network asynchrony.

在通常情况下，当网络强同步下，并且最高优先级块提议者是诚实的时，BA⋆达成最终共识通过使用其最后一步来确认在同一轮次中不存在任何其他正在被商定的块。 否则，BA⋆可能会宣布**初步共识**，如果它不能确认由于可能的网络不同步造成的其他块的缺失。

A key aspect of BA⋆’s design is that it keeps no secrets, except for user private keys. This allows any user observing the messages to “passively participate” in the protocol: verify signatures, count votes, and reach the agreement decision.

**BA⋆设计的一个关键方面是它除了用户私钥之外没有任何秘密。** 这允许**任何观察消息的用户**“被动参与”协议：验证签名，计票并达成协议决定。

### 7.1 Main procedure of BA⋆

7.1  BA⋆的主要步骤

The top-level procedure implementing BA⋆, as invoked by Algorand, is shown in Algorithm 3. The procedure takes a context ctx, which captures the current state of the ledger, a round number, and a new proposed block, from the highest- priority block proposer (§6). Algorand is responsible for ensuring that the block is valid (by checking the proposed block’s contents and using an empty block if it is invalid, as described in §8). The context consists of the seed for sortition, the user weights, and the last agreed-upon block.

在算法3中显示了由Algorand调用的实现BA⋆的顶层步骤。该过程需要一个上下文ctx，这个上下文ctx从最高优先级区块提议者中捕获了帐本的当前状态，轮次编号和一个新提议的块(§6)。 Algorand负责确保块的有效性（通过检查建议的块的内容并使用空白块，如果它是无效的，如§8所述）。上下文ctx由用于抽签的种子，用户权益和最新正在商议的块组成。

For efficiency, BA⋆ votes for hashes of blocks, instead of entire block contents. At the end of the BA⋆ algorithm, we use the BlockOfHash() function to indicate that, if BA⋆ has not yet received the pre-image of the agreed-upon hash, it must obtain it from other users (and, since the block was agreed upon, many of the honest users must have received it during block proposal).

**为了提高效率，BA⋆投票选择块的哈希值，而不是整个块的内容。** 在BA⋆算法结束时，我们使用BlockOfHash()函数来表明，如果BA⋆尚未收到正商议的hash的原象，则它必须从其他用户处获得它（并且由于块在被商定中，许多诚实的用户一定在区块提议阶段就收到了）。

The BA⋆ algorithm also determines whether it established final or tentative consensus. We will discuss this check in detail when we discuss Algorithm 8.

BA⋆算法也决定它是建立最终的还是暂定的共识。 我们将在讨论算法8时详细讨论这个地方。

![image](https://user-images.githubusercontent.com/5023721/43708479-1aa004d4-999d-11e8-915f-450bbd1c6b42.png)

Algorithm 3: Running BA⋆ for the next round, with a proposed block. H is a cryptographic hash function.

![image](https://user-images.githubusercontent.com/5023721/43708505-2a2c1262-999d-11e8-93b7-2ba483290df9.png)

Algorithm 4: Voting for value by committee members. user.sk and user.pk are the user’s private and public keys.

### 7.2 Voting

### 7.2 投票

**Sending votes (Algorithm 4).** Algorithm 4 shows the pseudocode for CommitteeVote(), which checks if the user is selected for the committee in a given round and step of BA⋆. The CommitteeVote() procedure invokes Sortition() from Algorithm 1 to check if the user is chosen to partici- pate in the committee. If the user is chosen for this step, the user gossips a signed message containing the value passed to CommitteeVote(), which is typically the hash of some block. To bind the vote to the context, the signed message includes the hash of the previous block.

**发送投票(算法4)。**算法4展示 CommitteeVote()的伪代码，它检查一个用户在BA⋆给定的轮次和步骤中是否被选为委员会成员。CommitteeVote()过程从算法1调用Sortition()来检查用户是否被选择参与委员会。 如果用户在此步骤被选中，则这个用户gossip一个包含传递给CommitteeVote()的value的签名消息，该值通常是**某个块的散列值**。 为了将投票绑定到上下文，签名的消息包含前一个块的散列。

**Counting votes (Algorithm 5 and Algorithm 6).** The CountVotes() procedure (Algorithm 5) reads messages that belong to the current round and step from the incomingMsgs buffer. (For simplicity, our pseudocode assumes that a back- ground procedure takes incoming votes and stores them into that buffer, indexed by the messages’ round and step.) It pro- cesses the votes by calling the ProcessMsg() procedure for every message (Algorithm 6), which ensures that the vote is valid. Note that no private state is required to process these messages.

**计算投票(算法5和算法6)。**CountVotes() 过程（算法5）读取属于当前一轮的消息，并基于incomingMsgs 缓冲区。 （为了简单起见，我们的伪代码假定后台程序获取传入的投票并将它们存储到缓冲区中，并由消息的循环和步骤进行索引。）它通过调用每个消息的ProcessMsg()过程来处理投票（算法6）， 这确保了投票的有效性。 请注意，不需要私有状态来处理这些消息。

ProcessMsg() returns not just the value contained in the message, but also the number of votes associated with that value. If the message was not from a chosen committee member, ProcessMsg() returns zero votes. If the committee member was chosen several times (see §5), the number of votes returned by ProcessMsg() reflects that as well. Pro- cessMsg() also returns the sortition hash, which we will use later in Algorithm 9.

ProcessMsg()不仅返回消息中包含的值value，**还返回与该value关联的投票数**。 如果消息不是来自选定的委员会成员，ProcessMsg()会返回零票。 如果委员会成员被多次选中（见§5），ProcessMsg()返回的选票数量也反映了这一点。 ProcessMsg()也返回抽签hash，我们将在后面的算法9中使用它。

As soon as one value has more than T · τ votes, CountVotes() returns that value. τ is the expected num- ber of users that Sortition() selects for the committee, and is the same for each step (τ step ) with the exception of the final step (τ final ). T is a fraction of that expected committee size (T > 3 2 ) that defines BA⋆’s voting threshold; this is also the same for every step except the final step, and we analyze it in §7.5. If not enough messages were received within the allo- cated λ time window, then CountVotes() produces timeout. The threshold ensures that if one honest user’s CountVotes() returns a particular value, then all other honest users will return either the same value or timeout, even under the weak synchrony assumption (see Lemma 1 in Appendix C.2).

只要一个值超过T·τ票，CountVotes()就会返回该值。 τ是Sortition()为委员会选择的用户的期望数量，对于每个步骤（τ\_step）都是相同的，但最后一步（τ\_final）除外。 T是预期委员会规模（T> 2/3）的一部分，定义BA⋆的投票阈值; 除了最后一步之外，每个步骤都是一样的，我们在§7.5中对它进行分析。 如果在分配的λ时间窗口内没有收到足够的消息，则CountVotes()会产生TIMEOUT。 阈值确保了如果一个诚实用户的CountVotes()返回一个特定值，那么即使在弱同步假设下，所有其他诚实用户也会返回相同的值或超时值（参见附录C.2中的引理1）

![image](https://user-images.githubusercontent.com/5023721/43719559-9e072c1c-99c0-11e8-8f75-b372a2e54a86.png)

Algorithm 5: Counting votes for round and step.

![image](https://user-images.githubusercontent.com/5023721/43719587-acc2fa38-99c0-11e8-8904-63f76c56a9b7.png)

Algorithm 6: Validating incoming vote message m.

### 7.3 Reduction

### 7.3 简化

The Reduction() procedure, shown in Algorithm 7, converts the problem of reaching consensus on an arbitrary value (the hash of a block) to reaching consensus on one of two values: either a specific proposed block hash, or the hash of an empty block. Our reduction is inspired by Turpin and Coan’s two-step technique [50]. This reduction is important to ensure liveness.

在算法7中显示的Reduction()过程将在任意值（块的hash）上达成一致的问题转换为在两个值中的一个上达成共识：要么是特定的建议块hash，要么是空块的hash。 Turpin和Coan的两步法[50]启发我们的简化Reduction。 这种简化对于确保活力liveness非常重要。

In the first step of the reduction, each committee member votes for the hash of the block passed to Reduction() by BA⋆(). In the second step, committee members vote for the hash that received at least T ·τ votes in the first step, or the hash of the default empty block if no hash received enough votes. Reduction() ensures that there is at most one non-empty block that can be returned by Reduction() for all honest users.

在化简的第一步中，每个委员会成员投票通过BA⋆()传递给Reduction()的块的hash。 在第二步中，委员会成员投票选择在第一步中至少得到T·τ票的hash，或者如果没有hash获得足够票数，则默认空块的hash。 Reduction()确保所有诚实用户最多只有一个可由Reduction()返回的非空块。

In the common case when the network is strongly syn- chronous and the highest-priority block proposer was hon- est, most (e.g., 95%) of the users will call Reduction() with the same hblock parameter, and Reduction() will return that same hblock result to most users as well.

在通常情况下当网络是强同步的时候，且最高优先级的出块者是诚实的，大多数(例如95%)用户会用同样的hblock作为参数调用Reduction()，且Reduction()会返回同样的hblock的结果。

![image](https://user-images.githubusercontent.com/5023721/43721036-51af3950-99c4-11e8-9fcd-c4084634f567.png)

Algorithm 7: The two-step reduction.
On the other hand, if the highest-priority block proposer was dishonest, different users may start Reduction() with different hblock parameters. In this case, no single hblock value may be popular enough to cross the threshold of votes. As a result, Reduction() will return empty_hash.

另一方面，如果最高优先级的出块者是非诚实的，不同的用户可能会得到不同的hblock来开始Reduction()，在这种条件下，没有单一的hblock能够获得阈值下的投票，结果导致Reduction()会返回empty_hash。

### 7.4 Binary agreement

### 二元商定

Algorithm 8 shows BinaryBA⋆(), which reaches consensus on one of two values: either the hash passed to BinaryBA⋆() or the hash of the empty block. BinaryBA⋆() relies on Re- duction() to ensure that at most one non-empty block hash is passed to BinaryBA⋆() by all honest users.

算法8展示了BinaryBA⋆()，它在两个值中的一个上达成共识：传递给BinaryBA⋆()的hash或空块的hash。 BinaryBA⋆()依赖Reduction()来确保所有诚实用户至多将一个非空块哈希传递给BinaryBA⋆()。

**Safety with strong synchrony.** In each step of BinaryBA⋆(), a user who has seen more than T · τ votes for some value will vote for that same value in the next step (if selected). However, if no value receives enough votes, BinaryBA⋆() chooses the next vote in a way that ensures consensus in a strongly synchronous network.

**强同步下的安全性。**在BinaryBA⋆()的每一步中，如果一个用户看到了超过T·τ的某个 value(hash)，那么他将在下一步（如果选中）投票给相同的 value(hash)。 但是，如果没有任何value获得足够的投票，BinaryBA⋆() 会在某种程度上选择下一个投票，以确保在强同步网络中达成一致。

Specifically, user A may receive votes from an adversary that push the votes observed by A past the threshold, but the adversary might not send the same votes to other users (or might not send them in time). As a result, A returns consensus on a block, but other users timed out in that step. It is crucial that BinaryBA⋆() chooses the votes for the next step in a way that will match the block already returned by A. Algorithm 8 follows this rule: every return statement is coupled with a check for timeout that sets the next-step vote to the same value that could have been returned.

具体而言，用户A可以接收来自攻击者的投票，该投票会推动由A观察到的投票超过阈值，**但攻击者可能不向其他用户发送相同的投票（或者可能不及时发送）**。 因此，A会返回块的共识，但其他用户在该步骤中超时。 BinaryBA⋆() 选择下一步的投票是非常重要的，它将匹配A已经返回的块。算法8遵循以下规则：每个返回语句都与一个超时检查相结合，以设置下一步投票给可能已经返回的相同value。

It is also crucial that BinaryBA⋆() is able to collect enough votes in the next step to carry forward the value that A already reached consensus on. If there are many users like A that have already returned consensus, BinaryBA⋆() may not have enough users to push CountVotes() in the next step past the threshold. To avoid this problem, whenever a user returns consensus, that user votes in the next three steps with the value they reached consensus on.

BinaryBA⋆()能够在下一步收集足够的选票以发挥A已达成共识的价值也是至关重要的。 如果有许多像A这样的用户已经返回了共识。BinaryBA⋆()可能没有足够的用户在超过阈值的下一步中调用CountVotes()。 为了避免这个问题，每当用户返回共识时，该用户在接下来的三个步骤中用他们达成一致的value进行投票。

![image](https://user-images.githubusercontent.com/5023721/43724242-115e87fe-99cc-11e8-993c-7ea405870dba.png)

Algorithm 8: BinaryBA⋆ executes until consensus is reached on either block_hash or empty_hash.

In the common case, when the network is strongly syn- chronous and the block proposer was honest, BinaryBA⋆() will start with the same block_hash for most users, and will reach consensus in the first step, since most committee mem- bers vote for the same block_hash value.

在通常情况下，当网络强烈同步，并且块提议者是诚实的时，BinaryBA⋆()将为大多数用户开始相同的block_hash，并且将在第一步达成共识，因为大多数委员会成员投票给相同的block_hash 值

**Safety with weak synchrony.** If the network is not strongly synchronous (e.g., there is a partition), BinaryBA⋆() may return consensus on two different blocks. For example, suppose that, in the first step of BinaryBA⋆(), all users vote for block_hash, but only one honest user, A, receives those votes. In this case, A will return consensus on block_hash, but all other users will move on to the next step. Now, the other users vote for block_hash again, because CountVotes() returned timeout. However, let’s assume the network drops all of these votes. Finally, the users vote for empty_hash in the third step, the network becomes well behaved, and all votes are delivered. As a result, the users will keep vot- ing for empty_hash until the next iteration of the loop, at which point they reach consensus on empty_hash. This is undesirable because BinaryBA⋆() returned consensus on two different hashes to different honest users.

**弱同步的安全性。**如果网路不是强同步(如网络分区), BinaryBA⋆()或许会返回2个不同的区块，例如，在 BinaryBA⋆()的第一步中，所有的用户都投票给了block_hash，但是只有1个诚实用户，A收到了那些投票，在这种情况下，A返回了bloch_hash的共识，但是其他的所有用户已经走到了下一步了，现在其他用户重新开始投block_hash，因为CountVotes() 返回了TIMEOUT。然后，我们假定网络丢弃了所有的投票，最后，用户为empty_hash发起投票在第3阶段，网络变好了，然后所有的投票都延后了，作为结果，用户直到下一个循环之前都在为empty_hash投票，在这一点上他们为empty_hash达成了共识。这是不可取的因为BinaryBA⋆()在不同的诚实用户上达成了2个不同hash的共识。

BA⋆() addresses this problem by introducing the notion of final and tentative consensus. Final consensus means that BA⋆() will not reach consensus on any other block for that round. Tentative consensus means that BA⋆() was unable to guarantee safety, either because of network asynchrony or due to a malicious block proposer.

BA⋆()通过引入最终和暂定共识的概念来解决这个问题。 最终共识意味着BA⋆()不会在该轮的任何其他区块达成共识。 暂时的共识意味着BA⋆()无法保证安全，无论是因为网络异步还是由于恶意阻止提议者。

BA⋆() designates consensus on value V as “final” if BinaryBA⋆() reached consensus on V in the very first step, and if enough users observed this consensus being reached. Specifically, BinaryBA⋆() sends out a vote for the special final step to indicate that a user reached consensus on some value in the very first step, and BA⋆() collects these votes to determine whether final consensus was achieved. In a strongly synchronous network with an honest block pro- poser, BinaryBA⋆() will reach consensus in the first step, most committee members will vote for the consensus block in the special final step in BinaryBA⋆(), and will receive more than a threshold of such votes in BA⋆(), thus declaring the block as final. The final step is analogous to the final confirmation step implemented in many Byzantine-resilient protocols [15, 34]. 

BA⋆()指定在 value V 上达到共识作为 "final" 如果BinaryBA⋆()在**第一步**中达成了对V的共识，并且有足够的用户观察到达成了这个共识。 具体而言，BinaryBA⋆()发出了对特殊的FINAL步骤的投票，以表明用户在第一步中达成了对某个value的共识，BA⋆()收集这些投票以确定是否达成最终共识。 在一个诚实的块提议者的强同步网络中，BinaryBA⋆()将在第一步中达成共识，大多数委员会成员将在BinaryBA⋆()中的特殊FINAL步骤中投票选出共识块，并且将获得超过阈值的像在BA⋆()中的选票，因此宣布该区块为最终。 FINAL步骤类似于许多拜占庭弹性协议中实施的最终确认步骤[15,34]。

Intuitively, this guarantees safety because a large thresh- old of users have already declared consensus for V , and will not vote for any other value in the same round. In our ex- ample above, where user A reached consensus on a different block than all other users, neither block would be designated as final, because only one user (namely, A) observed consen- sus at the first step, and there would never be enough votes to mark that block as final. Appendix C.1 formalizes and proves this safety property.

直观地说，这保证了安全性，因为超过阈值的用户已经宣布了V的共识，并且不会在同一轮中投票给其他任何value。 在我们上面的例子中，用户A在不同于所有其他用户的块上达成一致意见时，这两个块都不会被指定为最终的，因为只有一个用户（即A）在第一步观察到共识，并且永远不会有足够的选票 将该块标记为最终。 附录C.1正式化并证明了这种安全特性。

One subtle issue arises due to the fact that BA⋆ relies on a committee to declare final consensus, instead of relying on all participants. As a result, even if one user observes final consensus, an adversary that controls the network may be able to prevent a small fraction of other users from reaching any kind of consensus (final or tentative) for an arbitrary number of steps. Each of these steps give the adversary an additional small probability of reaching consensus on a different value (e.g., the empty block). To bound the total probability of an adversary doing so, BA⋆ limits the total number of allowed steps; Appendix C.1 relies on this. If the protocol runs for more than MaxSteps steps, BA⋆ halts with- out consensus and relies on the recovery protocol described in §8.2 to recover liveness.

由于BA⋆依靠委员会宣布最终共识，而不是依靠所有参与者，所以出现了一个微妙的问题。 因此，即使一个用户观察到最终共识，控制网络的对手也许能够阻止一小部分其他用户达成任意类型的共识（最终或暂定），以实现任意数量的步骤。 这些步骤中的每一个都使得对手在不同的值（例如，空白块）上达成共识的附加小概率。 为了限制对手这样做的总概率，BA⋆限制了允许步数的总数; 附录C.1依赖于此。 如果协议运行超过MaxSteps步骤，则BA⋆将停止而不会达成共识，并依靠§8.2中描述的恢复协议恢复活跃性。

![image](https://user-images.githubusercontent.com/5023721/43726094-c9b480c0-99d0-11e8-9344-e4ab5c4d2c6b.png)

Algorithm 9: Computing a coin common to all users.

**Getting unstuck.** One remaining issue is that consensus could get stuck if the honest users are split into two groups, A and B, and the users in the two groups vote for different values (say, we are in step 1, A votes for empty_hash, and B votes for block_hash). Neither group is large enough to gather enough votes on their own, but together with the adversary’s votes, group A is large enough. In this situation, the adversary can determine what every user will vote for in the next step. To make some user vote for empty_hash in the next step, the adversary sends that user the adversary’s own votes for empty_hash just before the timeout expires, which, together with A’s votes, crosses the threshold. To make the user vote for block_hash, the adversary does not send any votes to that user; as a result, that user’s CountVotes() will return timeout, and the user will choose block_hash for the next step’s vote, according to the BinaryBA⋆() algorithm. This way, the adversary can split the users into two groups in the next step as well, and continue this attack indefinitely.

**变得紊乱。** 剩下的一个问题是，如果诚实用户被分成两组，A和B，并且两组中的用户投票给不同的值（例如，我们在步骤1中，A为empty_hash投票，B投票给block_hash）。这两个团体都不够大，无法单独收集足够的选票，但是和攻击者的选票的时候，groupA的就足够大了。在这种情况下，攻击者可以确定每个用户在下一步将投票的内容。为了让一些用户在下一步中为empty_hash投票，攻击者在超时过期之前将该攻击者的投票发送给empty_hash，这种情况下与A的投票一起超过阈值。要让用户对block_hash投票，攻击者不会向该用户发送任何投票; 因此，根据BinaryBA⋆()算法，该用户的CountVotes()将返回TIMEOUT，并且用户将为下一步的投票选择block_hash。这样，对手也可以在下一步将用户分成两组，并且无限期地继续进行攻击。

The attack described above requires the adversary to know how a user will vote after receiving timeout from CountVotes(). The third step of BinaryBA⋆() is designed to avoid this attack by pushing towards accepting either block_hash or empty_hash based on a random “common coin,” meaning a binary value that is predominantly the same for all users. Although this may sound circular, the users need not reach formal consensus on this common coin. As long as enough users observe the same coin bit, and the bit was not known to the attacker in advance of the step, BinaryBA⋆() will reach consensus in the next iteration of the loop with probability 1/2 (i.e., the probability that the attacker guessed wrong). By repeating these steps, the probability of consen- sus quickly approaches 1.

上述攻击要求攻击者知道从CountVotes()收到TIMEOUT后用户会如何投票。 BinaryBA⋆()的第三步旨在通过推动接受基于随机“普通硬币”来选出block_hash或empty_hash来避免这种攻击，这意味着对所有用户而言主要是相同的二进制value。虽然这听起来可能是循环的，但用户无需就这个普通硬币达成正式共识。只要有足够的用户观察(投掷出)到相同的硬币位，并且该位在该步骤之前并未被攻击者知道，则BinaryBA⋆()将在循环的下一次迭代中以概率1/2达到一致（即概率攻击者猜测错误）。通过重复这些步骤，共识的可能性很快接近1。

To implement this coin we take advantage of the VRF- based committee member hashes attached to all of the mes- sages. Every user sets the common coin to be the least- significant bit of the lowest hash it observed in this step, as shown in Algorithm 9. If a user gets multiple votes (i.e., several of their sub-users were selected), then Common- Coin() considers multiple hashes from that user, by hashing that user’s sortition hash with the sub-user index. Notice that hashes are random (since they are produced by hashing the pseudo-random VRF output), so their least-significant bits are also random. The common coin is used only when CountVotes() times out, giving sufficient time for all votes to propagate through the network. If the committee member with the lowest hash is honest, then all users that received his message observe the same coin.

为了实现这个硬币，我们利用所有消息附带的基于VRF的委员会成员散列。 如算法9所示，每个用户将普通硬币设置为在此步骤中观察到的最低散列值的最低位。如果用户获得多票（即，选择了几个其子用户），则CommonCoin() 通过将该用户的抽签哈希与子用户索引进行哈希来考虑该用户的多个散列。 注意哈希是随机的（因为它们是通过散列伪随机VRF输出产生的），所以它们的最低有效位也是随机的。 常用硬币仅在CountVotes()超时时使用，给予所有选票在网络中传播的足够时间。 如果委员会成员中哈希值最低的人是诚实，那么接收到他的消息的所有用户都会观察到同一枚硬币。

If a malicious committee-member happens to hold the lowest hash, then he might send it to only some users. This may result in users observing different coin values, and thus will not help in reaching consensus. However, since sor- tition hashes are pseudo-random, the probability that an honest user has the lowest hash is h (the fraction of money held by honest users), and thus there is at least an h > 2/3 probability that the lowest sortition hash holder will be hon- est, which leads to consensus with probability 1/2 ·h > 1/3 at each loop iteration. This allows Appendix C.3 to show that, with strong synchrony, BA⋆ does not exceed MaxSteps with overwhelming probability.

如果一个恶意的会员会成员刚好持有最小值哈希，那么他可能会把其发送给其中一些用户，这可能会导致这些用户看到了不同的硬币值，这对达成共识没有帮助，然后因为抽签哈希是伪随机数，一个诚实用户有最低hash的可能性是h，只要至少h>2/3的可能性，最低抽签hash的持有者就会是诚实的，这回导致共识的可能性 1/2 * h > 1/3 在每一次循环迭代中。附录C.3 展示了在强网络同步中，BA⋆会以压倒性的概率不超过MAXSTEPS

### 7.5 Committee size

### 7.5 委员会大小

The fraction h > 2/3 of weighted honest users in Algorand must translate into a “sufficiently honest” committee for BA⋆. BA⋆ has two parameters at its disposal: τ, which con- trols the expected committee size, and T, which controls the number of votes needed to reach consensus (T ·τ). We would like T to be as small as possible for liveness, but the smaller T is, the larger τ needs to be, to ensure that an adversary does not obtain enough votes by chance. Since a larger com- mittee translates into a higher bandwidth cost, we choose two different parameter sets: T final and τ final for the final step, which ensures an overwhelming probability of safety regardless of strong synchrony, and T step and τ step for all other steps, which achieve a reasonable trade-off between liveness, safety, and performance.

h>2/3权重占比的诚实用户一定被翻译成BA⋆ 中的“足够诚实”的委员会成员。BA⋆有两个参数可供选择：控制预期委员会规模的τ和控制达成共识（T*τ）所需投票数量的T(0<T<1)。 我们希望T越小越好，但T越小，τ就越大，以确保对手不会偶然获得足够的选票。 由于较大的委员会会导致较高的带宽成本，因此我们选择两个不同的参数集：T\_final和τ\_final用于FINAL步，不管强同步性如何，确保绝对安全的可能性，以及用于所有其他步骤的T\_step和τ\_step， 活跃性，安全性和性能之间的合理平衡。

![image](https://user-images.githubusercontent.com/5023721/43726378-94d811ea-99d1-11e8-88a2-bef923f83e6a.png)

To make the constraints on τ step and T step precise, let us denote the number of honest committee members by д and the malicious ones byb; in expectation, b +д =τ step , butb +д can vary since it is chosen by sortition. To ensure liveness, as we prove in Appendix C.2, BA⋆ requires 2 1 д +b ≤ T step ·τ step and д > T step ·τ step .

为了使τ\_step和T\_step的约束更加精确，我们使用g表示诚实委员会的成员的数量，用b表示攻击者的数量，在表达式中b +g=τ\_step，但b +g可以变化，因为它是通过抽签来选择的。 为了确保活力，我们在附录C.2中证明，BA⋆要求1/2 *g+b≤T\_step·τ_step 且 g>T\_step·τ\_step。

Due to the probabilistic nature of how committee members are chosen, there is always some small chance that the b and д for some step fail to satisfy the above constraints, and BA⋆’s goal is to make this probability negligible. Figure 3 plots the expected committee size τ step that is needed to satisfy both constraints, as a function ofh, for a probability of violation of 5×10 −9 ; Appendix B describes this computation in more detail. The figure shows a trade-off: the weaker the assumption on the fraction of money held by honest users (h), the larger the committee size needs to be. The results show that, as h approaches 2/3 , the committee size grows quickly. However, at h = 80%, τ step = 2,000 can ensure that these constraints hold with probability 1 − 5 × 10 −9 (using T step = 0.685).

由于选择委员会成员的概率性质，某些步骤中的b和g总是有一些小的机会不能满足上述约束条件，BA⋆的目标是使这个概率可以忽略不计。图3绘制了满足这两个约束所需的预期委员会规模τ\_step，作为h的函数，对于违反5×10-9的概率;附录B更详细地描述了这个计算。该数字显示了一个折衷：对诚实用户持有的部分货币的假设（h）越小，委员会规模就越大。结果显示，随着h接近2/3，委员会规模迅速增长。然而，在h = 80％时，τ\_step= 2,000可以确保这些约束以1-5×10-9的概率保持（使用T\_step = 0.685）。

The constraints on τ final and T final are dictated by the proof of safety under weak synchrony; Appendix C.1 shows that τ final = 10,000 suffices with T final = 0.74.

τ\_final和T\_final的约束是由弱同步下的安全性证明决定的;附录C.1显示τfinal= 10,000，T\_final = 0.74。

With these parameters, BA⋆ ensures safety even if the lowest-priority block proposer is malicious (proposes dif- ferent blocks). Appendix C provides proofs of BA⋆’s safety under weak synchrony (§C.1), liveness under strong syn- chrony (§C.2), and efficiency (§C.3).

有了这些参数，BA⋆确保即使安全最低优先级的块提议者是恶意的（提出不同的块）。附录C提供了BA weak在弱同步（§C.1），强同步（§C.2）下的活性和效率（§C.3）下的安全性的证据。