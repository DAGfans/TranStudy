>* **Source：** [https://eprint.iacr.org/2018/104.pdf](https://eprint.iacr.org/2018/104.pdf)  
>* **TranStudy：** [https://github.com/DAGfans/TranStudy/edit/master/Papers/PHANTOM/4-SCALABILITY%20AND%20NETWORK%20DELAYS.md](https://github.com/DAGfans/TranStudy/edit/master/Papers/PHANTOM/4-SCALABILITY%20AND%20NETWORK%20DELAYS.md)


# 4. SCALABILITY AND NETWORK DELAYS

# 4. 可扩展性和网络延迟

## A. The propagation delay parameter $D_{max}$

## A. 传播时延参数$D_{max}$

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

## B. The anticone size parameter k

## B. 反锥体大小参数k

The parameter k is decided from the outset and hard-coded in the protocol. It is defined as
follows:

协议在开头就定义并硬编码了参数k。定义如下所示：
$$
 k(D_{max},δ) := min \left \{\hat{k} \in N : (1-e^{-2\cdot D_{max}\cdot λ})^{-1}\cdot \left ( \sum_{j=\hat{k} + 1}^{∞}{e^{-2\cdot D_{max}\cdot λ}\cdot \frac{(2\cdot D_{max}\cdot λ)^j}{j!}}\right )\right \}
$$


The motivation here is to devise a bound over the number of blocks created in parallel. Since
the block creation rate follows a Poisson process, for an arbitrary block B created at time t,
at most $k(D_{max},δ)$ additional blocks were created in the time interval$[t - D_{max}, t + D_{max}]$, with
probability of at least $1 - δ$. (In more detail: The second multiplicand in the definition of k
bounds the probability that more than k blocks were created in parallel to B in the time interval
$[t - D_{max}, t + D_{max}]$. This term is divided by $1 - e^{-2 \cdot D_{max}}$, which is the probability that
at least one block was created during this time, namely, B. Thus, conditioned on the
appearance of B, at most k blocks were created during this time interval, with probability of
$1 - δ$ at least.)

这里的目的是为同时创建的区块数量设计一个上限。由于区块的创建频率服从泊松分布，因此对任意一个在时刻t创建的区块B，在时间间隔$[t - D_{max}, t + D_{max}]$,中创建出的其它区块数量最多为$k(D_{max},δ)$，且其概率至少为 $1 - δ$。（更具体地说：在k的定义中的第二个乘数限定了在时间间隔$[t - D_{max}, t + D_{max}]$中在B以外同时创建的区块数量大于k的概率。这一项被$1 - e^{-2 \cdot D_{max}}$，也就是在此期间至少有一个区块即B被创建的概率所除。因此，以B的出现为条件，在此期间最多有k个区块被创建的概率至少为$1 - δ$。）

Observe that blocks created in the intervals $[0, t - D_{max})$ and $(t + D_{max}, ∞)$, by honest nodes,
belong to B’s past and future sets, respectively. Consequently, in principle, $|anticone(B)| ≤ k$
with probability of $1 - δ$ at least. However, an attacker can artificially increase B’s anticone by
creating blocks that do not reference it and by withholding his blocks so that B cannot reference
them.

我们可以观察到在$[0, t - D_{max})$和$(t + D_{max}, ∞)$期间内由诚实节点创建的区块分别属于B的过去和将来集。因此，原则上$|anticone(B)| ≤ k$的概率至少为$1 - δ$。尽管如此，攻击者可以通过创建不引用B的区块，并且将区块隐藏起来不让B指向它们，从而人为增大B的反锥体。


## C. Trade-offs

## C. 取舍

Theorem 5, and the parameterization of PHANTOM in (1), tie between $k$, $D_{max}$, $λ$, and $δ$.
Striving for a better performance by modifying one parameter (e.g., increasing $λ$ to obtain larger
throughput and more frequent blocks) must be understood and considered against the effect on
all other parameters.

定理5，以及PHANTOM的公式（1）将$k$、$D_{max}$、$λ$和$δ$这几个参数相互联系了起来。如果想修改某个参数从而获取更好的性能（比如，增大$λ$从而获得更大的吞吐量和更频繁的区块创建次数），就必须理解并考虑对其它参数的影响。

**Increased block creation rate.** Although the security threshold does not deteriorate as $λ$ is
increased, $λ$ cannot be increased indefinitely, or otherwise the network becomes congested. The
value of $λ$ should be set such that nodes that are expected to participate in the system can
support such a throughput. For instance, if nodes are required to maintain a bandwidth of at
least 1MB per second, and blocks are of size $b$ = 1MB, then the block creation rate should
be set to $λ$ = 1 blocks per second (this is merely a back-of-the-envelope calculation, and in
practice other messages consume the bandwidth as well).

**更高的区块创建频率。** 虽然安全阈值并不会随着$λ$增大而变差，但是$λ$并不能无穷增大，不然网络就会变得拥堵不堪。$λ$的值所对应的吞吐量必须能够被期望加入系统的节点所支撑。比如，如果我们要求节点保持至少每秒1MB的带宽，并且区块的大小是$b$ = 1MB，那么区块创建频率就必须设为$λ$ = 每秒1个区块（这只是粗略的计算，实际中还会有其它消息消耗带宽）。

**Higher security threshold.** Theorem 5 states the security threshold in terms of $δ$. Following (1)
we notice that tightening the security threshold - by choosing a lower $δ$ - requires increasing
$k$. A large $k$ leads to slow confirmation times, as will be discussed shortly.
(The advanced reader should notice that although increasing $λ$ has a similar negative effect on $k$, it has at the
same time a positive effect on confirmation times, and so a certain $λ$ will be optimal as far as confirmation times are concerned.)

**更高的安全阈值。** 定理5通过$δ$描述了安全阈值。从（1）我们可以注意到，（通过选择一个较小的$δ$值从而）收紧安全阈值就需要增大$k$。$k$值增大就意味着确认时间变慢，下文很快就会讨论到。（高阶读者应该能够意识到$λ$对$k$也有类似的负面影响，同时对确认时间有正面影响，因此只要能考虑到确认时间，$λ$的值必然就是最佳的。）

**Larger safety margin.** Similarly, if $D_{max}$ is to be increased, one needs to increase $k$ as well
in order to maintain the same security level (represented by $δ$).

**更大的安全边界。** 类似地，如果要增大$D_{max}$，为了维持安全等级（由$δ$表示）不变，就需要增大$k$。

As discussed in Subsection 4.1, it is better to overestimate $D$ and choose a large $D_{max}$ in order
remain on the safe side. (Several blockchain based projects do not do so, and consequently compromise the security threshold of their system.) 
Recall that the security of Bitcoin’s chain depends on the assumption
that $D \cdot λ \ll 1$ , namely, that w.h.p. at least $D$ seconds pass between consecutive blocks, so that
forks are rare. Thus, Bitcoin’s large safety margin over $D$ suppresses its throughput severely as
it requires selecting a very low block rate $λ = 1 / 600$ (one block per 10 minutes). This is not
the case with PHANTOM’s DAG, as the security of the DAG ordering does not rely on the
assumption $D \cdot λ \ll 1$. Therefore, even if we overestimate $D$, we can still allow for very high
block creation rates while maintaining the same level of security. Consequently, PHANTOM
supports a very large throughput, and does not suffer from a security-scalability tradeoff.

我们在4.1节讨论过，一般最好是高估$D$并选择一个较大的$D_{max}$从而确保安全性。（有的基于区块链的项目并不这么做，因此它们的系统在安全阈值方面做了妥协。）回顾一下，比特币的安全性依赖于$D \cdot λ \ll 1$这个假设，也就是说，一个区块被创建之后，极有可能要至少过$D$秒才会有下一个区块被创建，因此分叉极少。因此，比特币的大于$D$的安全边界严重抑制了它的吞吐量，因为它要选择一个很低的区块创建频率$λ = 1 / 600$（每10分钟一个区块）。而PHANTOM的DAG并不是这样，因为DAG排序的安全性并不依赖$D \cdot λ \ll 1$这个假设。因此，即使我们高估了$D$，也依然可以在维持安全等级不变的情况下允许十分高的区块创建频率。所以PHANTOM支持非常大的吞吐量，并且不需要对安全性和可扩展性做出取舍。

That said, in PHANTOM there is still a tradeoff between a large safety margin and fast
convergence of the protocol. A gross overestimation of $D_{max}$ - resulting an increase in $k$ -
would significantly slow down the waiting time for transaction settlement. Thus, $D_{max}$ should
be set to a reasonable level. In Section 7 we discuss how this tradeoff can be restricted to visible
conflicts only, and how applications such as *Payments* can enjoy very fast confirmation times
nonetheless.

即便如此，在PHANTOM中依然需要在大的安全边界和协议的快速收敛之间进行取舍。对$D_{max}$过高的估计——从而导致$k$的增大——会使交易结算的等待时间显著变慢。因此，$D_{max}$的值需要设置为一个合理的水平。在第7节我们会讨论如何能够将这个取舍限制在可见的冲突内，以及在这种情况下*支付*之类的应用如何维持很快的确认时间。
