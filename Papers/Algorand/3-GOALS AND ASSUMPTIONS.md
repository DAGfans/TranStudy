
## 3 GOALS AND ASSUMPTIONS

Algorand allows users to agree on an ordered log of transactions, and achieves two goals with respect to the log:

**Safety goal.** With overwhelming probability, all users agree on the same transactions.
 More precisely, if one honest user accepts transaction A (i.e., it appears in the log), then any future transactions accepted by other honest users will appear in a log that already contains A.
 This holds even for isolated users that are disconnected from the network—e.g., by Eclipse attacks [28].


**Liveness goal.** In addition to safety, Algorand also makes progress (i.e., allows new transactions to be added to the log) under additional assumptions about network reachability that we describe below.
 Algorand aims to reach consensus on a new set of transactions within roughly one minute.


**Assumptions.
** Algorand makes standard cryptographic assumptions such as public-key signatures and hash functions.
 Algorand assumes that honest users run bug-free software.
 As mentioned earlier, Algorand assumes that the fraction of money held by honest users is above some threshold h (a constant greater than 2/3), but that an adversary can participate in Algorand and own some money.
 We believe that this assumption is reasonable, since it means that in order to successfully attack Algorand, the attacker must invest substantial financial resources in it.
 Algorand assumes that an adversary can corrupt targeted users, but that an adversary cannot corrupt a large number of users that hold a significant fraction of the money (i.e., the amount of money held by honest, non-compromised users must remain over the threshold).


To achieve liveness, Algorand makes a “strong synchrony” assumption that most honest users (e.g., 95%) can send messages that will be received by most other honest users (e.g., 95%) within a known time bound.
 This assumption allows the adversary to control the network of a few honest users, but does not allow the adversary to manipulate the network at a large scale, and does not allow network partitions.


Algorand achieves safety with a “weak synchrony” assumption: the network can be asynchronous (i.e., entirely controlled by the adversary) for a long but bounded period of time (e.g., at most 1 day or 1 week).
 After an asynchrony period, the network must be strongly synchronous for a reasonably long period again (e.g., a few hours or a day) for Algorand to ensure safety.
 More formally, the weak synchrony assumption is that in every period of length b (think of b as a day or a week), there must be a strongly synchronous period of length s < b (an s of a few hours suffices).


Finally, Algorand assumes loosely synchronized clocks across all users (e.g., using NTP) in order to recover liveness after weak synchrony.
 Specifically, the clocks must be close enough in order for most honest users to kick off the recovery protocol at approximately the same time.
 If the clocks are out of sync, the recovery protocol does not succeed.
