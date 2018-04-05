
#### 4. SCALABILITY AND NETWORK DELAYS

#### 4. 可扩展性和网络延迟

A. The propagation delay parameter Dmax

A. 传播时延参数Dmax

The scalability of a distributed algorithm is closely tied to the assumptions it makes on the
underlying network, and specifically on its propagation delay D. The real value of D is both
unknown and sensitive to shifting network conditions. For this reason, Bitcoin operates under
the assumption that D is much smaller than 10 minutes, and sets the average block interval
time to 10 minutes. While this seems like an overestimation of the network’s propagation delay
under normal conditions (at least in 2018’s Internet terms), some safety margin must be taken,
to account for peculiar network conditions as well. Similarly, in PHANTOM we assume that
the unknown D is upper bounded by some Dmax which is known to the protocol. The protocol
does not explicitly encode Dmax, rather, it is parameterized with k which depends on it, as will
be described in the next subsection.

一个分布式算法的可扩展性与其对底层网络所作的假设——尤其是传播时延D密切相关。D的真实数值是未知的，并且会随着网络条件的时刻变化而改变。因此，比特币假设D远小于10分钟，并将区块的平均生成间隔定为10分钟。虽然这看起来高估了正常条件下的网络传播时延（至少以2018年的因特网来说），但我们必须设置一些安全界限，以应对特殊罕见的网络条件。类似地，在PHANTOM中，虽然D的值未知，但我们假设其上限是Dmax，而这个值对协议来说是已知的。协议并不会显式地对Dmax进行编码，而是用参数k来表示它。也就是说，k的值取决于Dmax。下一小节将会对此进行论述。

The use of an *a priori* known bound Dmax distinguishes PHANTOM’s security model from
that of SPECTRE [8]. While the security of both protocols depends on the assumption that the
network’s propagation delay D is upper bounded by some constant, in SPECTRE the value of
such a constant need not be known or assumed by the protocol, whereas PHANTOM makes
explicit use of this parameter (via k) when ordering the DAG’s blocks. The fact that the order
between any two blocks becomes robust in PHANTOM, but not in SPECTRE, should be ascribed
to this added assumption; see further discussion in Section 7.

PHANTOM的安全模型与SPECTRE的不同就在于使用了先验已知的上限Dmax。虽然两者协议的安全性都建立在网络传播时延D的上限是某个常数这个假设上，但在SPECTRE中协议不需要知道或假设该常数的数值，而PHANTOM在对DAG里的区块排序时（通过k）显式地使用了这个参数。PHANTOM中任意两个区块之间的顺序都是鲁棒的，而SPECTRE则不是，正是因为PHANTOM比SPECTRE多做了这一步假设；见第7节进一步的讨论。

B. The anticone size parameter k

B. 反锥体大小参数k

The parameter k is decided from the outset and hard-coded in the protocol. It is defined as
follows:

协议在开头就定义并硬编码了参数k。定义如下所示：

```
k(Dmax,δ) := min {\hat{k} \in N : (1 − e^{-2 ·Dmax·λ})^{-1} · ($\sum_{j=\hat{k} + 1}^{\infty} {e^{−2 ·Dmax·λ} · (2·Dmax·λ)^j / j!}) < δ} (1)
```

The motivation here is to devise a bound over the number of blocks created in parallel. Since
the block creation rate follows a Poisson process, for an arbitrary block B created at time t,
at most k(Dmax,δ) additional blocks were created in the time interval[t - Dmax, t + Dmax], with
probability of at least 1 - δ. (In more detail: The second multiplicand in the definition of k
bounds the probability that more than k blocks were created in parallel to B in the time interval
[t - Dmax, t + Dmax]. This term is divided by 1 - e^{-2 ·Dmax·λ}, which is the probability that
at least one block was created during this time, namely, B. Thus, conditioned on the
appearance of B, at most k blocks were created during this time interval, with probability of
1 - δ at least.)

这里的目的是为同时创建的区块数量设计一个上限。由于区块的创建频率服从泊松分布，因此对任意一个在时刻t创建的区块B，在时间间隔[t - Dmax, t + Dmax]中创建出的其它区块数量最多为k(Dmax,δ)，且其概率至少为1 - δ。（更具体地说：在k的定义中的第二个乘数限定了在时间间隔[t - Dmax, t + Dmax]中在B以外同时创建的区块数量大于k的概率。这一项被1 - e^{-2 ·Dmax·λ}，也就是在此期间至少有一个区块即B被创建的概率所除。因此，以B的出现为条件，在此期间最多有k个区块被创建的概率至少为1 - δ。）

Observe that blocks created in the intervals [0, t - Dmax) and (t + Dmax, ∞), by honest nodes,
belong to B’s past and future sets, respectively. Consequently, in principle, |anticone(B)| ≤ k
with probability of 1 - δ at least. However, an attacker can artificially increase B’s anticone by
creating blocks that do not reference it and by withholding his blocks so that B cannot reference
them.

我们可以观察到在[0, t - Dmax)和(t + Dmax, ∞)期间内由诚实节点创建的区块分别属于B的过去和将来集。因此，原则上|anticone(B)| ≤ k的概率至少为1 - δ。尽管如此，攻击者可以通过创建不引用B的区块，并且将区块隐藏起来不让B指向它们，从而人为增大B的反锥体。


C. Trade-offs

Theorem 5, and the parameterization of PHANTOM in (1), tie betweenk,Dmax,λ, andδ.
Striving for a better performance by modifying one parameter (e.g., increasingλto obtain larger
throughput and more frequent blocks) must be understood and considered against the effect on
all other parameters.

Increased block creation rate.Although the security threshold does not deteriorate asλis
increased,λcannot be increased indefinitely, or otherwise the network becomes congested. The
value ofλ should be set such that nodes that are expected to participate in the system can
support such a throughput. For instance, if nodes are required to maintain a bandwidth of at
least 1MBper second, and blocks are of sizeb= 1MB, then the block creation rate should
be set toλ= 1blocks per second (this is merely a back-of-the-envelope calculation, and in
practice other messages consume the bandwidth as well).

Higher security threshold.Theorem 5 states the security threshold in terms ofδ. Following (1)
we notice that tightening the security threshold – by choosing a lowerδ– requires increasing
k. A largekleads to slow confirmation times, as will be discussed shortly.^11

Larger safety margin.Similarly, ifDmaxis to be increased, one needs to increasekas well
in order to maintain the same security level (represented byδ).
As discussed in Subsection 4.1, it is better to overestimateDand choose a largeDmaxin order
remain on the safe side.^12 Recall that the security of Bitcoin’s chain depends on the assumption
thatD·λ 1 , namely, that w.h.p. at leastDseconds pass between consecutive blocks, so that
forks are rare. Thus, Bitcoin’s large safety margin overDsuppresses its throughput severely as
it requires selecting a very low block rateλ= 1/ 600 (one block per 10 minutes). This is not
the case with PHANTOM’s DAG, as the security of the DAG ordering does not rely on the
assumptionD·λ 1. Therefore, even if we overestimateD, we can still allow for very high
block creation rates while maintaining the same level of security. Consequently, PHANTOM
supports a very large throughput, and does not suffer from a security-scalability tradeoff.
That said, in PHANTOM there is still a tradeoff between a large safety margin and fast
convergence of the protocol. A gross overestimation ofDmax– resulting an increase ink–
would significantly slow down the waiting time for transaction settlement. Thus,Dmaxshould
be set to a reasonable level. In Section 7 we discuss how this tradeoff can be restricted to visible
conflicts only, and how applications such asPaymentscan enjoy very fast confirmation times
nonetheless.
