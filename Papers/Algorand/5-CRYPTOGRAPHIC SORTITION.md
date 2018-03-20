## 5 CRYPTOGRAPHIC SORTITION

Cryptographic sortition is an algorithm for choosing a random subset of users according to per-user weights; that is, given a set of weights w i and the weight of all users W = i w i , the probability that user i is selected is proportional to w i /W .
 The randomness in the sortition algorithm comes from a publicly known random seed; we describe later how this seed is chosen.
 To allow a user to prove that they were chosen, sortition requires each user i to have a public/private key pair, (pk i ,sk i ).


![figure 2](https://user-images.githubusercontent.com/22833166/37686063-d9220766-2cd0-11e8-9e8f-7d867fb9720d.jpg)

**Figure 2:** An overview of one step of BA⋆.
 To simplify the figure, each user is shown twice: once at the top of the diagram and once at the bottom.
 Each arrow color indicates a message from a particular user.


Sortition is implemented using verifiable random functions (VRFs) [38].
 Informally, on any input string x, VRF sk (x) returns two values: a hash and a proof.
 The hash is a hashlenbit-long value that is uniquely determined by sk and x, but is indistinguishable from random to anyone that does not know sk.
 The proof π enables anyone that knows pk to check that the hash indeed corresponds to x, without having to know sk.
 For security, we require that the VRF provides these properties even if pk and sk are chosen by an attacker.


### 5.1 Selection procedure

Using VRFs, Algorand implements cryptographic sortition as shown in Algorithm 1.
 Sortition requires a role parameter that distinguishes the different roles that a user may be selected for; for example, the user may be selected to propose a block in some round, or they may be selected to be the member of the committee at a certain step of BA⋆.
 Algorand specifies a threshold τ that determines the expected number of users selected for that role.

Algorithm 1: The cryptographic sortition algorithm.


It is important that sortition selects users in proportion to their weight; otherwise, sortition would not defend against Sybil attacks.
 One subtle implication is that users may be chosen more than once by sortition (e.g., because they have a high weight).
 Sortition addresses this by returning the j parameter, which indicates how many times the user was chosen.
 Being chosen j times means that the user gets to participate as j different “sub-users.
”

To select users in proportion to their money, we consider each unit of Algorand as a different “sub-user.
” If user i owns w i (integral) units of Algorand, then simulated user (i,j) with j ∈ {1,...,w i } represents the j th unit of currency τ i owns, and is selected with probability p = W , where W is the total amount of currency units in Algorand.


As shown in Algorithm 1, a user performs sortition by computing ⟨hash,π⟩ ← VRF sk (seed||role), where sk is the user’s secret key.
 The pseudo-random hash determines how many sub-users are selected, as follows.
 The probability that exactly k out of the w (the user’s weight) sub-users are selected follows the binomial distribution,  Í B(k;w,p) =w k p k (1−p) w−k , where w k=0 B(k;w,p) = 1.
 Since B(k 1 ;n 1 ,p) + B(k 2 ;n 2 ,p) = B(k 1 + k 2 ;n 1 + n 2 ,p), splitting a user’s weight (currency) among Sybils does not affect the number of selected sub-users under his/her control.


To determine how many of a user’s w sub-users are selected, the sortition algorithm divides the interval [0,1) into consecutive intervals of the form I j = h Í Í  j j+1

B(k;w,p), B(k;w,p) for j ∈ {0,1,...,w}. If k=0 k=0

hash/2 hashlen (where hashlen is the bit-length of hash) falls in the interval I j , then the user has exactly j selected sub-users.
 The number of selected sub-users is publicly verifiable using the proof π (from the VRF output).


Sortition provides two important properties.
 First, given a random seed, the VRF outputs a pseudo-random hash value, which is essentially uniformly distributed between 0 and 2 hashlen −1.
 As a result, users are selected at random based on their weights.
 Second, an adversary that does not know sk i cannot guess how many times user i is chosen, or if i was chosen at all (more precisely, the adversary cannot guess any better than just by randomly guessing based on the weights).


The pseudocode for verifying a sortition proof, shown in Algorithm 2, follows the same structure to check if that user was selected (the weight of the user’s public key is obtained from the ledger).
 The function returns the number of selected sub-users (or zero if the user was not selected at all).


Algorithm 2: Pseudocode for verifying sortition of a user with public key pk.


### 5.2 Choosing the seed

Sortition requires a seed that is chosen at random and publicly known.
 For Algorand, each round requires a seed that is publicly known by everyone for that round, but cannot be controlled by the adversary; otherwise, an adversary may be able to choose a seed that favors selection of corrupted users.


In each round of Algorand a new seed is published.
 The seed published at Algorand’s round r is determined using VRFs with the seed of the previous round r −1.
 More specifically, during the block proposal stage of round r −1, every user u selected for block proposal also computes a proposed seed for round r as ⟨seed r ,π⟩ ← VRF sk u (seed r−1 ||r).
 Algorand requires that sk u be chosen by u well in advance of the seed for that round being determined (§5.
3).
 This ensures that even if u is malicious, the resulting seed r is pseudo-random.


This seed (and the corresponding VRF proof π) is included in every proposed block, so that once Algorand reaches agreement on the block for round r −1, everyone knows seed r at the start of round r.
 If the block does not contain a valid seed (e.g., because the block was proposed by a malicious user and included invalid transactions), users treat the entire proposed block as if it were empty, and use a cryptographic hash function H (which we assume is a random oracle) to compute the associated seed for round r as seed r = H(seed r−1 ||r).
 The value of seed 0 , which bootstraps seed selection, can be chosen at random at the start of Algorand by the initial participants (after their public keys are declared) using distributed random number generation [14].


To limit the adversary’s ability to manipulate sortition, and thus manipulate the selection of users for different committees, the selection seed (passed to Algorithm 1 and Algorithm 2) is refreshed once every R rounds.
 Namely, at roundr Algorand calls the sortition functions with seed r−1−(r mod R) .


### 5.3 Choosing sk u well in advance of the seed

Computing seed r requires that every user’s secret key sk u is chosen well in advance of the selection seed used in that round, i.e., seed r−1−(r mod R) .
 When the network is not strongly synchronous, the attacker has complete control over the links, and can therefore drop block proposals and force users to agree on empty blocks, such that future selection seeds can be computed.
 To mitigate such attacks Algorand relies on the weak synchrony assumption (in every period of length b, there must be a strongly synchronous period of length s < b).
 Whenever Algorand performs cryptographic sortition for round r, it checks the timestamp included in the agreed-upon block for round r −1−(r mod R), and uses the keys (and associated weights) from the last block that was created b-time before block r −1−(r mod R).
 The lower bound s on the length of a strongly synchronous period should allow for sufficiently many blocks to be created in order to ensure with overwhelming probability that at least one block was honest.
 This ensures that, as long as s is suitably large, an adversary u choosing a key sk u cannot predict the seed for round r.


This look-back periodb has the following trade-off: a large b mitigates the risk that attackers are able break the weak synchronicity assumption, yet it increases the chance that users have transferred their currency to someone else and therefore have nothing to lose if the system’s security breaks.
 This is colloquially known as the “nothing at stake” problem; one possible way to avoid this trade-off, which we do not explore in Algorand, is to take the minimum of a user’s current balance and the user’s balance from the look-back block as the user’s weight.


Appendix A formally analyzes the number of blocks that Algorand needs to be created in the period s when the network is strongly connected.
 We show that to ensure a small probability of failure F, the number of blocks is logarith1 mic in F , which allows us to obtain high security with a reasonably low number of required blocks.

