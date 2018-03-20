6

BLOCK PROPOSAL

To ensure that some block is proposed in each round, Al- gorand sets the sortition threshold for the block-proposal role, τ proposer , to be greater than 1 (although Algorand will reach consensus on at most one of these proposed blocks). Appendix B proves that choosing τ proposer = 26 ensures that a reasonable number of proposers (at least one, and no more than 70, as a plausible upper bound) are chosen with very high probability (e.g., 1−10 −11 ).

Minimizing unnecessary block transmissions. One risk of choosing several proposers is that each will gossip their own proposed block. For a large block (say, 1 MByte), this can incur a significant communication cost. To reduce this cost, the sortition hash is used to prioritize block propos- als: For each selected sub-user 1,...,j of user i, the priority for the block proposal is obtained by hashing the (verifiably random) hash output of VRF concatenated with the sub-user index. The highest priority of all the block proposer’s se- lected sub-users is the priority of the block.

Algorand users discard messages about blocks that do not have the highest priority seen by that user so far. Algorand also gossips two kinds of messages: one contains just the priorities and proofs of the chosen block proposers (from sortition), and the other contains the entire block, which also includes the proposer’s sortition hash, and proof. The first kind of message is small (about 200 Bytes), and propagates quickly through the gossip network. These messages enable most users to learn who is the highest priority proposer, and thus quickly discard other proposed blocks.

Waiting for block proposals. Each user must wait a cer- tain amount of time to receive block proposals via the gossip protocol. Choosing this time interval does not impact Algo- rand’s safety guarantees but is important for performance. Waiting a short amount of time will mean no received pro- posals. If the user receives no block proposals, he or she initializes BA⋆ with the empty block, and if many users do so, Algorand will reach consensus on an empty block. On the other hand, waiting too long will receive all block proposals but also unnecessarily increase the confirmation latency.

To determine the appropriate amount of time to wait for block proposals, we consider the plausible scenarios that a user might find themselves in. When a user starts waiting for block proposals for roundr, they may be one of the first users to reach consensus in round r −1. Since that user completed round r −1, sufficiently many users sent a message for the last step of BA⋆ in that round, and therefore, most of the network is at most one step behind this user. Thus, the user must somehow wait for others to finish the last step of BA⋆ from round r − 1. At this point, some proposer in round r that happens to have the highest priority will gossip their priority and proof message, and the user must somehow wait to receive that message. Then, the user can simply wait until they receive the block corresponding to the highest priority proof (with a timeout λ block , on the order of a minute, after which the user will fall back to the empty block).

It is impossible for a user to wait exactly the correct amount for the first two steps of the above scenario. Thus, Algorand estimates these quantities (λ stepvar , the variance in how long it takes different users to finish the last step of BA⋆, and λ priority , the time taken to gossip the priority and proof message), and waits for λ stepvar +λ priority time to identify the highest priority. §10 experimentally shows that these parameters are, conservatively, 5 seconds each. As mentioned above, Algorand would remain safe even if these estimates were inaccurate.

Malicious proposers. Even if some block proposers are malicious, the worst-case scenario is that they trick different Algorand users into initializing BA⋆ with different blocks. This could in turn cause Algorand to reach consensus on an empty block, and possibly take additional steps in doing so. However, it turns out that even this scenario is relatively unlikely. In particular, if the adversary is not the highest pri- ority proposer in a round, then the highest priority proposer will gossip a consistent version of their block to all users. If the adversary is the highest priority proposer in a round, they can propose the empty block, and thus prevent any real transactions from being confirmed. However, this happens with probability of at most 1−h, by Algorand’s assumption that at least h > 2/3 of the weighted user are honest.

7

BA⋆

The execution of BA⋆ consists of two phases. In the first phase, BA⋆ reduces the problem of agreeing on a block to agreement on one of two options. In the second phase, BA⋆ reaches agreement on one of these options: either agreeing on a proposed block, or agreeing on an empty block.

Each phase consists of several interactive steps; the first phase always takes two steps, and the second phase takes two steps if the highest-priority block proposer was honest (sent the same block to all users), and as we show in our analysis an expected 11 steps in the worst case of a malicious highest-priority proposer colluding with a large fraction of committee participants at every step.

In each step, every committee member casts a vote for some value, and all users count the votes. Users that receive more than a threshold of votes for some value will vote for that value in the next step (if selected as a committee member). If the users do not receive enough votes for any value, they time out, and their choice of vote for the next step is determined by the step number.

In the common case, when the network is strongly syn- chronous and the highest-priority block proposer was hon- est, BA⋆ reaches final consensus by using its final step to confirm that there cannot be any other agreed-upon block in the same round. Otherwise, BA⋆ may declare tentative consensus if it cannot confirm the absence of other blocks due to possible network asynchrony.

A key aspect of BA⋆’s design is that it keeps no secrets, except for user private keys. This allows any user observing the messages to “passively participate” in the protocol: verify signatures, count votes, and reach the agreement decision.

7.1 Main procedure of BA⋆

The top-level procedure implementing BA⋆, as invoked by Algorand, is shown in Algorithm 3. The procedure takes a context ctx, which captures the current state of the ledger, a round number, and a new proposed block, from the highest- priority block proposer (§6). Algorand is responsible for ensuring that the block is valid (by checking the proposed block’s contents and using an empty block if it is invalid, as described in §8). The context consists of the seed for sortition, the user weights, and the last agreed-upon block.

For efficiency, BA⋆ votes for hashes of blocks, instead of entire block contents. At the end of the BA⋆ algorithm, we use the BlockOfHash() function to indicate that, if BA⋆ has not yet received the pre-image of the agreed-upon hash, it must obtain it from other users (and, since the block was agreed upon, many of the honest users must have received it during block proposal).

The BA⋆ algorithm also determines whether it established final or tentative consensus. We will discuss this check in detail when we discuss Algorithm 8.

procedure BA⋆(ctx,round,block):

hblock ← Reduction(ctx,round,H(block)) hblock ⋆ ← BinaryBA⋆(ctx,round,hblock) // Check if we reached “final” or “tentative” consensus r ← CountVotes(ctx,round,final,T final ,τ final ,λ step ) if hblock ⋆ = r then

return ⟨final,BlockOfHash(hblock ⋆ )⟩ else

return ⟨tentative,BlockOfHash(hblock ⋆ )⟩

Algorithm 3: Running BA⋆ for the next round, with a proposed block. H is a cryptographic hash function.

procedure CommitteeVote(ctx,round,step,τ,value):

// check if user is in committee using Sortition (Alg. 1) role ← ⟨“committee”,round,step⟩ ⟨sorthash,π,j⟩ ← Sortition(user.sk,ctx.seed,τ,role,

ctx.weight[user.pk],ctx.W ) // only committee members originate a message if j > 0 then

Gossip(⟨user.pk,Signed user.sk (round,step,

sorthash,π,H(ctx.last_block),value)⟩)

Algorithm 4: Voting for value by committee members. user.sk and user.pk are the user’s private and public keys.

7.2 Voting

Sending votes (Algorithm 4). Algorithm 4 shows the pseudocode for CommitteeVote(), which checks if the user is selected for the committee in a given round and step of BA⋆. The CommitteeVote() procedure invokes Sortition() from Algorithm 1 to check if the user is chosen to partici- pate in the committee. If the user is chosen for this step, the user gossips a signed message containing the value passed to CommitteeVote(), which is typically the hash of some block. To bind the vote to the context, the signed message includes the hash of the previous block.

Counting votes (Algorithm 5 and Algorithm 6). The CountVotes() procedure (Algorithm 5) reads messages that belong to the current round and step from the incomingMsgs buffer. (For simplicity, our pseudocode assumes that a back- ground procedure takes incoming votes and stores them into that buffer, indexed by the messages’ round and step.) It pro- cesses the votes by calling the ProcessMsg() procedure for every message (Algorithm 6), which ensures that the vote is valid. Note that no private state is required to process these messages.

ProcessMsg() returns not just the value contained in the message, but also the number of votes associated with that value. If the message was not from a chosen committee member, ProcessMsg() returns zero votes. If the committee member was chosen several times (see §5), the number of votes returned by ProcessMsg() reflects that as well. Pro- cessMsg() also returns the sortition hash, which we will use later in Algorithm 9.

As soon as one value has more than T · τ votes, CountVotes() returns that value. τ is the expected num- ber of users that Sortition() selects for the committee, and is the same for each step (τ step ) with the exception of the final step (τ final ). T is a fraction of that expected committee size (T > 3 2 ) that defines BA⋆’s voting threshold; this is also the same for every step except the final step, and we analyze it in §7.5. If not enough messages were received within the allo- cated λ time window, then CountVotes() produces timeout. The threshold ensures that if one honest user’s CountVotes() returns a particular value, then all other honest users will return either the same value or timeout, even under the weak synchrony assumption (see Lemma 1 in Appendix C.2).

Algorithm 5: Counting votes for round and step.

procedure ProcessMsg(ctx,τ,m): ⟨pk,signed_m⟩ ← m if VerifySignature(pk,signed_m) , OK then

return ⟨0,⊥,⊥⟩ ⟨round,step,sorthash,π,hprev,value⟩ ← signed_m // discard messages that do not extend this chain if hprev , H(ctx.last_block) then return ⟨0,⊥,⊥⟩; votes ← VerifySort(pk,sorthash,π,ctx.seed,τ,

⟨“committee”,round,step⟩,ctx.weight[pk],ctx.W ) return ⟨votes,value,sorthash⟩

Algorithm 6: Validating incoming vote message m.

7.3 Reduction

The Reduction() procedure, shown in Algorithm 7, converts the problem of reaching consensus on an arbitrary value (the hash of a block) to reaching consensus on one of two values: either a specific proposed block hash, or the hash of an empty block. Our reduction is inspired by Turpin and Coan’s two-step technique [50]. This reduction is important to ensure liveness.

In the first step of the reduction, each committee member votes for the hash of the block passed to Reduction() by BA⋆(). In the second step, committee members vote for the hash that received at least T ·τ votes in the first step, or the hash of the default empty block if no hash received enough votes. Reduction() ensures that there is at most one non-empty block that can be returned by Reduction() for all honest users.

In the common case when the network is strongly syn- chronous and the highest-priority block proposer was hon- est, most (e.g., 95%) of the users will call Reduction() with the same hblock parameter, and Reduction() will return that same hblock result to most users as well.

Algorithm 7: The two-step reduction.
On the other hand, if the highest-priority block proposer was dishonest, different users may start Reduction() with different hblock parameters. In this case, no single hblock value may be popular enough to cross the threshold of votes. As a result, Reduction() will return empty_hash.

7.4 Binary agreement

Algorithm 8 shows BinaryBA⋆(), which reaches consensus on one of two values: either the hash passed to BinaryBA⋆() or the hash of the empty block. BinaryBA⋆() relies on Re- duction() to ensure that at most one non-empty block hash is passed to BinaryBA⋆() by all honest users.

Safety with strong synchrony. In each step of BinaryBA⋆(), a user who has seen more than T · τ votes for some value will vote for that same value in the next step (if selected). However, if no value receives enough votes, BinaryBA⋆() chooses the next vote in a way that ensures consensus in a strongly synchronous network.

Specifically, user A may receive votes from an adversary that push the votes observed by A past the threshold, but the adversary might not send the same votes to other users (or might not send them in time). As a result, A returns consensus on a block, but other users timed out in that step. It is crucial that BinaryBA⋆() chooses the votes for the next step in a way that will match the block already returned by A. Algorithm 8 follows this rule: every return statement is coupled with a check for timeout that sets the next-step vote to the same value that could have been returned.

It is also crucial that BinaryBA⋆() is able to collect enough votes in the next step to carry forward the value that A already reached consensus on. If there are many users like A that have already returned consensus, BinaryBA⋆() may not have enough users to push CountVotes() in the next step past the threshold. To avoid this problem, whenever a user returns consensus, that user votes in the next three steps with the value they reached consensus on.

procedure BinaryBA⋆(ctx,round,block_hash):

step ← 1 r ← block_hash empty_hash ← H(Empty(round,H(ctx.last_block))) while step < MaxSteps do

CommitteeVote(ctx, round, step, τ step , r) r ← CountVotes(ctx,round,step,T step ,τ step ,λ step ) if r = timeout then r ← block_hash else if r , empty_hash then for step < s ′ ≤ step+3 do CommitteeVote(ctx, round, s ′ , τ step , r) if step = 1 then CommitteeVote(ctx, round, final, τ final , r) return r step++

CommitteeVote(ctx, round, step, τ step , r) r ← CountVotes(ctx,round,step,T step ,τ step ,λ step )

if r = timeout then r ← empty_hash

else if r = empty_hash then for step < s ′ ≤ step+3 do CommitteeVote(ctx, round, s ′ , τ step , r) return r step++

CommitteeVote(ctx, round, step, τ step , r) r ← CountVotes(ctx,round,step,T step ,τ step ,λ step )

if r = timeout then

if CommonCoin(ctx,round,step,τ step ) = 0 then r ← block_hash else

votes. In this case, A will return consensus on block_hash, but all other users will move on to the next step. Now, the other users vote for block_hash again, because CountVotes() returned timeout. However, let’s assume the network drops all of these votes. Finally, the users vote for empty_hash in the third step, the network becomes well behaved, and all votes are delivered. As a result, the users will keep vot- ing for empty_hash until the next iteration of the loop, at which point they reach consensus on empty_hash. This is undesirable because BinaryBA⋆() returned consensus on two different hashes to different honest users.

BA⋆() addresses this problem by introducing the notion of final and tentative consensus. Final consensus means that BA⋆() will not reach consensus on any other block for that round. Tentative consensus means that BA⋆() was unable to guarantee safety, either because of network asynchrony or due to a malicious block proposer.

BA⋆() designates consensus on value V as “final” if BinaryBA⋆() reached consensus on V in the very first step, and if enough users observed this consensus being reached. Specifically, BinaryBA⋆() sends out a vote for the special final step to indicate that a user reached consensus on some value in the very first step, and BA⋆() collects these votes to determine whether final consensus was achieved. In a strongly synchronous network with an honest block pro- poser, BinaryBA⋆() will reach consensus in the first step, most committee members will vote for the consensus block in the special final step in BinaryBA⋆(), and will receive more than a threshold of such votes in BA⋆(), thus declaring the block as final. The final step is analogous to the final confirmation step implemented in many Byzantine-resilient protocols [15, 34].

r ← empty_hash

step++ // No consensus after MaxSteps; assume network // problem, and rely on §8.2 to recover liveness. HangForever()

Algorithm 8: BinaryBA⋆ executes until consensus is reached on either block_hash or empty_hash.
In the common case, when the network is strongly syn- chronous and the block proposer was honest, BinaryBA⋆() will start with the same block_hash for most users, and will reach consensus in the first step, since most committee mem- bers vote for the same block_hash value.

Safety with weak synchrony. If the network is not strongly synchronous (e.g., there is a partition), BinaryBA⋆() may return consensus on two different blocks. For example, suppose that, in the first step of BinaryBA⋆(), all users vote for block_hash, but only one honest user, A, receives those votes. In this case, A will return consensus on block_hash, but all other users will move on to the next step. Now, the other users vote for block_hash again, because CountVotes() returned timeout. However, let’s assume the network drops all of these votes. Finally, the users vote for empty_hash in the third step, the network becomes well behaved, and all votes are delivered. As a result, the users will keep vot- ing for empty_hash until the next iteration of the loop, at which point they reach consensus on empty_hash. This is undesirable because BinaryBA⋆() returned consensus on two different hashes to different honest users.

BA⋆() addresses this problem by introducing the notion of final and tentative consensus. Final consensus means that BA⋆() will not reach consensus on any other block for that round. Tentative consensus means that BA⋆() was unable to guarantee safety, either because of network asynchrony or due to a malicious block proposer.

BA⋆() designates consensus on value V as “final” if BinaryBA⋆() reached consensus on V in the very first step, and if enough users observed this consensus being reached. Specifically, BinaryBA⋆() sends out a vote for the special final step to indicate that a user reached consensus on some value in the very first step, and BA⋆() collects these votes to determine whether final consensus was achieved. In a strongly synchronous network with an honest block pro- poser, BinaryBA⋆() will reach consensus in the first step, most committee members will vote for the consensus block in the special final step in BinaryBA⋆(), and will receive more than a threshold of such votes in BA⋆(), thus declaring the block as final. The final step is analogous to the final confirmation step implemented in many Byzantine-resilient protocols [15, 34].
Intuitively, this guarantees safety because a large thresh- old of users have already declared consensus for V , and will not vote for any other value in the same round. In our ex- ample above, where user A reached consensus on a different block than all other users, neither block would be designated as final, because only one user (namely, A) observed consen- sus at the first step, and there would never be enough votes to mark that block as final. Appendix C.1 formalizes and proves this safety property.

One subtle issue arises due to the fact that BA⋆ relies on a committee to declare final consensus, instead of relying on all participants. As a result, even if one user observes final consensus, an adversary that controls the network may be able to prevent a small fraction of other users from reaching any kind of consensus (final or tentative) for an arbitrary number of steps. Each of these steps give the adversary an additional small probability of reaching consensus on a different value (e.g., the empty block). To bound the total probability of an adversary doing so, BA⋆ limits the total number of allowed steps; Appendix C.1 relies on this. If the protocol runs for more than MaxSteps steps, BA⋆ halts with- out consensus and relies on the recovery protocol described in §8.2 to recover liveness.

Algorithm 9: Computing a coin common to all users.
Getting unstuck. One remaining issue is that consensus could get stuck if the honest users are split into two groups, A and B, and the users in the two groups vote for different values (say, we are in step 1, A votes for empty_hash, and B votes for block_hash). Neither group is large enough to gather enough votes on their own, but together with the adversary’s votes, group A is large enough. In this situation, the adversary can determine what every user will vote for in the next step. To make some user vote for empty_hash in the next step, the adversary sends that user the adversary’s own votes for empty_hash just before the timeout expires, which, together with A’s votes, crosses the threshold. To make the user vote for block_hash, the adversary does not send any votes to that user; as a result, that user’s CountVotes() will return timeout, and the user will choose block_hash for the next step’s vote, according to the BinaryBA⋆() algorithm. This way, the adversary can split the users into two groups in the next step as well, and continue this attack indefinitely.

The attack described above requires the adversary to know how a user will vote after receiving timeout from CountVotes(). The third step of BinaryBA⋆() is designed to avoid this attack by pushing towards accepting either block_hash or empty_hash based on a random “common coin,” meaning a binary value that is predominantly the same for all users. Although this may sound circular, the users need not reach formal consensus on this common coin. As long as enough users observe the same coin bit, and the bit was not known to the attacker in advance of the step, BinaryBA⋆() will reach consensus in the next iteration of the loop with probability 1/2 (i.e., the probability that the attacker guessed wrong). By repeating these steps, the probability of consen- sus quickly approaches 1.

To implement this coin we take advantage of the VRF- based committee member hashes attached to all of the mes- sages. Every user sets the common coin to be the least- significant bit of the lowest hash it observed in this step, as shown in Algorithm 9. If a user gets multiple votes (i.e., several of their sub-users were selected), then Common- Coin() considers multiple hashes from that user, by hashing that user’s sortition hash with the sub-user index. Notice that hashes are random (since they are produced by hashing the pseudo-random VRF output), so their least-significant bits are also random. The common coin is used only when CountVotes() times out, giving sufficient time for all votes to propagate through the network. If the committee member with the lowest hash is honest, then all users that received his message observe the same coin.
Figure 3: The committee size, τ, sufficient to limit the proba- bility of violating safety to 5×10 −9 . The x-axis specifiesh, the weighted fraction of honest users. ⋆ marks the parameters selected in our implementation.
If a malicious committee-member happens to hold the lowest hash, then he might send it to only some users. This may result in users observing different coin values, and thus will not help in reaching consensus. However, since sor- tition hashes are pseudo-random, the probability that an honest user has the lowest hash is h (the fraction of money held by honest users), and thus there is at least an h > 3 2 probability that the lowest sortition hash holder will be hon- est, which leads to consensus with probability 2 1 ·h > 1 3 at each loop iteration. This allows Appendix C.3 to show that, with strong synchrony, BA⋆ does not exceed MaxSteps with overwhelming probability.

7.5 Committee size

The fraction h > 3 2 of weighted honest users in Algorand must translate into a “sufficiently honest” committee for BA⋆. BA⋆ has two parameters at its disposal: τ, which con- trols the expected committee size, and T, which controls the number of votes needed to reach consensus (T ·τ). We would like T to be as small as possible for liveness, but the smaller T is, the larger τ needs to be, to ensure that an adversary does not obtain enough votes by chance. Since a larger com- mittee translates into a higher bandwidth cost, we choose two different parameter sets: T final and τ final for the final step, which ensures an overwhelming probability of safety regardless of strong synchrony, and T step and τ step for all other steps, which achieve a reasonable trade-off between liveness, safety, and performance.

To make the constraints on τ step and T step precise, let us denote the number of honest committee members by д and the malicious ones byb; in expectation, b +д =τ step , butb +д can vary since it is chosen by sortition. To ensure liveness, as we prove in Appendix C.2, BA⋆ requires 2 1 д +b ≤ T step ·τ step and д > T step ·τ step .

Due to the probabilistic nature of how committee members are chosen, there is always some small chance that the b and д for some step fail to satisfy the above constraints, and BA⋆’s goal is to make this probability negligible. Figure 3 plots the expected committee size τ step that is needed to satisfy both constraints, as a function ofh, for a probability of violation of 5×10 −9 ; Appendix B describes this computation in more detail. The figure shows a trade-off: the weaker the assumption on the fraction of money held by honest users (h), the larger the committee size needs to be. The results show that, as h approaches 3 2 , the committee size grows quickly. However, at h = 80%, τ step = 2,000 can ensure that these constraints hold with probability 1 − 5 × 10 −9 (using T step = 0.685).

The constraints on τ final and T final are dictated by the proof of safety under weak synchrony; Appendix C.1 shows that τ final = 10,000 suffices with T final = 0.74.

With these parameters, BA⋆ ensures safety even if the lowest-priority block proposer is malicious (proposes dif- ferent blocks). Appendix C provides proofs of BA⋆’s safety under weak synchrony (§C.1), liveness under strong syn- chrony (§C.2), and efficiency (§C.3).
