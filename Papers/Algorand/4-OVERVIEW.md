## 4 OVERVIEW

Algorand requires each user to have a public key.
 Algorand maintains a log of transactions, called a blockchain.
 Each transaction is a payment signed by one user’s public key transferring money to another user’s public key.
 Algorand grows the blockchain in asynchronous rounds, similar to Bitcoin.
 In every round, a new block, containing a set of transactions and a pointer to the previous block, is appended to the blockchain.
 In the rest of this paper, we refer to Algorand software running on a user’s computer as that user.


Algorand users communicate through a gossip protocol.
 The gossip protocol is used by users to submit new transactions.
 Each user collects a block of pending transactions that they hear about, in case they are chosen to propose the next block, as shown in Figure 1.
 Algorand uses BA⋆ to reach consensus on one of these pending blocks.

![figure 1](https://user-images.githubusercontent.com/22833166/37685757-9872e650-2ccf-11e8-841f-cee2eb0b01e6.jpg)

Figure 1: An overview of transaction flow in Algorand.


BA⋆ executes in steps, communicates over the same gossip protocol, and produces a new agreed-upon block.
 BA⋆ can produce two kinds of consensus: final consensus and tentative consensus.
 If one user reaches final consensus, this means that any other user that reaches final or tentative consensus in the same round must agree on the same block value (regardless of whether the strong synchrony assumption held).
 This ensures Algorand’s safety, since this means that all future transactions will be chained to this final block (and, transitively, to its predecessors).
 Thus, Algorand confirms a transaction when the transaction’s block (or any successor block) reaches final consensus.
 On the other hand, tentative consensus means that other users may have reached tentative consensus on a different block (as long as no user reached final consensus).
 A user will confirm a transaction from a tentative block only if and when a successor block reaches final consensus.


BA⋆ produces tentative consensus in two cases.
 First, if the network is strongly synchronous, an adversary may, with small probability, be able to cause BA⋆ to reach tentative consensus on a block.
 In this case, BA⋆ will not reach consensus on two different blocks, but is simply unable to confirm that the network was strongly synchronous.
 Algorand will eventually (in a few rounds) reach final consensus on a successor block, with overwhelming probability, and thus confirm these earlier transactions.


The second case is that the network was only weakly synchronous (i.e., it was entirely controlled by the adversary, with an upper bound on how long the adversary can keep control).
 In this case, BA⋆ can reach tentative consensus on two different blocks, forming multiple forks.
 This can in turn prevent BA⋆ from reaching consensus again, because the users are split into different groups that disagree about previous blocks.
 To recover liveness, Algorand periodically invokes BA⋆to reach consensus on which fork should be used going forward.
 Once the network regains strong synchrony, this will allow Algorand to choose one of the forks, and then reach final consensus on a subsequent block on that fork.


We now describe how Algorand’s components fit together.


**Gossip protocol. ** Algorand implements a gossip network (similar to Bitcoin) where each user selects a small random set of peers to gossip messages to.
 To ensure messages cannot be forged, every message is signed by the private key of its original sender; other users check that the signature is valid before relaying it.
 To avoid forwarding loops, users do not relay the same message twice.
 Algorand implements gossip over TCP and weighs peer selection based on how much money they have, so as to mitigate pollution attacks.


**Block proposal (§6).** All Algorand users execute cryptographic sortition to determine if they are selected to propose a block in a given round.
 We describe sortition in §5, but at a high level, sortition ensures that a small fraction of users are selected at random, weighed by their account balance, and provides each selected user with a priority, which can be compared between users, and a proof of the chosen user’s priority.
 Since sortition is random, there may be multiple users selected to propose a block, and the priority determines which block everyone should adopt.
 Selected users distribute their block of pending transactions through the gossip protocol, together with their priority and proof.
 To ensure that users converge on one block with high probability, block proposals are prioritized based on the proposing user’s priority, and users wait for a certain amount of time to receive the block.


**Agreement using BA⋆ (§7).** Block proposal does not guarantee that all users received the same block, and Algorand does not rely on the block proposal protocol for safety.
 To reach consensus on a single block, Algorand uses BA⋆.
 Each user initializes BA⋆ with the highest-priority block that they received.
 BA⋆ executes in repeated steps, illustrated in Figure 2.
 Each step begins with sortition (§5), where all users check whether they have been selected as committee members in that step.
 Committee members then broadcast a message which includes their proof of selection.
 These steps repeat until, in some step of BA⋆, enough users in the committee reach consensus.
 (Steps are not synchronized across users; each user checks for selection as soon as he observes the previous step had ended.
) As discussed earlier, an important feature of BA⋆ is that committee members do not keep private state except their private keys, and so can be replaced after every step, to mitigate targeted attacks on them.


**Efficiency.** When the network is strongly synchronous, BA⋆ guarantees that if all honest users start with the same initial block (i.e., the highest priority block proposer was honest), then BA⋆ establishes final consensus over that block and terminates precisely in 4 interactive steps between users.
 Under the same network conditions, and in the worst case of a particularly lucky adversary, all honest users reach consensus on the next block within expected 13 steps, as analyzed in Appendix C.
