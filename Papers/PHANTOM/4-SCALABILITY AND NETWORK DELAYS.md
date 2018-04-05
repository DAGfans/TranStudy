4. SCALABILITY AND NETWORK DELAYS

A. The propagation delay parameter D max

The scalability of a distributed algorithm is closely tied to the assumptions it makes on the underlying network, and speciﬁcally on its propagation delay D. The real value of D is both unknown and sensitive to shifting network conditions. For this reason, Bitcoin operates under the assumption that D is much smaller than 10 minutes, and sets the average block interval time to 10 minutes. While this seems like an overestimation of the network’s propagation delay under normal conditions (at least in 2018’s Internet terms), some safety margin must be taken, to account for peculiar network conditions as well. Similarly, in PHANTOM we assume that the unknown D is upper bounded by some D max which is known to the protocol. The protocol does not explicitly encode D max , rather, it is parameterized with k which depends on it, as will be described in the next subsection.

The use of an a priori known bound D max distinguishes PHANTOM’s security model from that of SPECTRE [8]. While the security of both protocols depends on the assumption that the network’s propagation delay D is upper bounded by some constant, in SPECTRE the value of such a constant need not be known or assumed by the protocol, whereas PHANTOM makes explicit use of this parameter (via k ) when ordering the DAG’s blocks. The fact that the order between any two blocks becomes robust in PHANTOM, but not in SPECTRE, should be ascribed to this added assumption; see further discussion in Section 7.

B. The anticone size parameter k

The parameter k is decided from the outset and hard-coded in the protocol. It is deﬁned as follows:

k(D

max

, δ) :=     ∞    X (2 · D max · λ) j  −1 1 − e

(1)

min k ˆ ∈ N : −2·D max ·λ ·  e −2·D max ·λ ·  < δ  j!  j=k+1 The motivation here is to devise a bound over the number of blocks created in parallel. Since the block creation rate follows a Poisson process, for an arbitrary block B created at time t, at most k(D max , δ) additional blocks were created in the time interval [t − D max , t + D max ], with probability of at least 1 − δ . 10

Observe that blocks created in the intervals [0, t−D max ) and (t+D max , ∞), by honest nodes, belong to B ’s past and future sets, respectively. Consequently, in principle, |anticone (B)| ≤ k with probability of 1 − δ at least. However, an attacker can artiﬁcially increase B ’s anticone by creating blocks that do not reference it and by withholding his blocks so that B cannot reference them.

10

In more detail: The second multiplicand in the deﬁnition of k bounds the probability that more than k blocks were created in parallel to B in the time interval [t − D max , t + D max ]. This term is divided by 1 − e −2·D max ·λ , which is the probability that at least one block was created during this time, namely, B. Thus, conditioned on the appearance of B, at most k blocks were created during this time interval, with probability of 1 − δ at least. 15

C. Trade-offs

Theorem 5, and the parameterization of PHANTOM in (1), tie between k , D max , λ, and δ . Striving for a better performance by modifying one parameter (e.g., increasing λ to obtain larger throughput and more frequent blocks) must be understood and considered against the effect on all other parameters.

Increased block creation rate. Although the security threshold does not deteriorate as λ is increased, λ cannot be increased indeﬁnitely, or otherwise the network becomes congested. The value of λ should be set such that nodes that are expected to participate in the system can support such a throughput. For instance, if nodes are required to maintain a bandwidth of at least 1 MB per second, and blocks are of size b = 1 MB, then the block creation rate should be set to λ = 1 blocks per second (this is merely a back-of-the-envelope calculation, and in practice other messages consume the bandwidth as well).

Higher security threshold. Theorem 5 states the security threshold in terms of δ . Following (1) we notice that tightening the security threshold – by choosing a lower δ – requires increasing k . A large k leads to slow conﬁrmation times, as will be discussed shortly. 11 Larger safety margin. Similarly, if D max is to be increased, one needs to increase k as well in order to maintain the same security level (represented by δ ).

As discussed in Subsection 4.1, it is better to overestimate D and choose a large D max in order remain on the safe side. 12 Recall that the security of Bitcoin’s chain depends on the assumption that D · λ  1, namely, that w.h.p. at least D seconds pass between consecutive blocks, so that forks are rare. Thus, Bitcoin’s large safety margin over D suppresses its throughput severely as it requires selecting a very low block rate λ = 1/600 (one block per 10 minutes). This is not the case with PHANTOM’s DAG, as the security of the DAG ordering does not rely on the assumption D · λ  1. Therefore, even if we overestimate D, we can still allow for very high block creation rates while maintaining the same level of security. Consequently, PHANTOM supports a very large throughput, and does not suffer from a security-scalability tradeoff.

That said, in PHANTOM there is still a tradeoff between a large safety margin and fast convergence of the protocol. A gross overestimation of D max – resulting an increase in k – would signiﬁcantly slow down the waiting time for transaction settlement. Thus, D max should be set to a reasonable level. In Section 7 we discuss how this tradeoff can be restricted to visible conﬂicts only, and how applications such as Payments can enjoy very fast conﬁrmation times nonetheless.
