> Source: https://www.cs.huji.ac.il/~yoni_sompo/pubs/15/inclusive_full.pdf

# 3 Security

# 3 安全

The original security analysis of Satoshi ([10]), as well as analysis done by others
[12, 13], has considered the probability of a successful double-spend attack
under the regular non-inclusive scheme. An alternative analysis may instead
measure the cost of the attack rather than their success probability (both have
been analyzed in [12]).

中本聪（[10]）最初的安全分析，
还有一些其他人的分析[12, 13]，
已经考虑了常规的非Inclusive体制下一个双花攻击成功的概率。
还有一种分析方式是衡量攻击的成本，
而非它们的成功概率（[12]对这两种方式都做了分析）。

Below we prove that the Inclusive version of the protocol is at least as secure
as the non-inclusive one, in terms of the probability of successful attacks. In
addition, we show that the cost of an attack under Inclusive can be made high,
by properly modifying the acceptance policy.

下面来证明Inclusive版本的协议在攻击成功概率上的安全性不会低于非Inclusive的版本。
另外我们还会说明，
Inclusive可以通过修改接受策略使一次攻击的成本变高。

## 3.1 Acceptance Policy

## 3.1 接受策略

The recipient of a given transaction observes the network’s published blocks,
and needs to decide when to consider the payment “accepted”, that is, when it
is safe to release the goods or services paid for by the transaction. He does so by
making sure his transaction is included and confirmed by the main chain, and
calculating the probability that it would be later excluded from it.

一个给定交易的接收方会观察网络中已经发布的区块，
并需要决定何时考虑“接受”付款，
也就是说，
何时为已付款的交易释放产品或服务才是安全的。
他要确保他的交易被主链所包含和确认，
并计算交易以后被剔除的概率。

**Probability of Successful Attacks** We now compare the probability of a
successful attack under the regular longest-chain protocol to the one under its
Inclusive version. Our method can apply to other main chain rules as well (e.g.,
GHOST). Recall that under Inclusive the blocks form a DAG, whereas when
Inclusive is not implemented they form a tree (see Sect. 2). Notice that if
G(t) is the block DAG at time t then if the network would have followed the noninclusive
setup, its block tree T(t) would be precisely the subgraph of
G(t) obtained by removing all edges in blocks’ reference list apart from the main
edges (i.e., the first pointer in every block). For any DAG
G let F(G) be its main chain according to the underlying selection rule
F (G can also be a tree).

**攻击成功的概率**
现在比较在常规最长链协议下与在Inclusive版本下的攻击成功概率。
这个方法也可以应用于其他主链规则（例如GHOST）。
回顾一下，
在Inclusive下，
区块形成一个有向无环图；
而当Inclusive没有被实现时它们形成一棵树（见第2节）。
注意，
如果G(t)是处于时刻t的区块图，
那么如果网络遵循的是非Inclusive规则，
它的区块树T(t)正好就是移除区块引用列表中除主边（即每个区块中的第一个指针）以外的所有边而得到的G(t)的子图。
对于任意一个有向无环图G，
令F(G)为根据基础选择规则F得到的主链（G也可以是一棵树）。

**Theorem 2.** Let G(t) be the block DAG at time t, and let T(t) be the block
tree that is obtained from G(t) by discarding the non-main edges. For any block
B ∈ F(G(t)),

**定理2：**
设G(t)为时刻t的区块图，
并设T(t)为丢弃非主边从而由G(t)得到的区块树。
对任意B ∈ F(G(t))，

<div align="center">
<img src="https://latex.codecogs.com/svg.latex?\forall&space;s&space;>&space;t:&space;\Pr(B&space;\notin&space;F(G(s)))&space;=&space;\Pr(B&space;\notin&space;F(T(s)))&space;\hfill&space;(2)" title="\forall s > t: \Pr(B \notin F(G(s))) = \Pr(B \notin F(T(s))) \hfill (2)" />
</div>

*Proof.* This is immediate from the fact that Inclusive does not change the way
the main chain is selected, therefore, for all s: F(G(s)) = F(T(s)). 
<span>&#9744;</span>

*证明：*
从Inclusive并不改变主链挑选方式的事实可以立即得出该结论。
因此对于所有的s都有：
F(G(s)) = F(T(s))。
<span>&#9744;</span>

As a corollary, the probability that a transaction would be excluded from the
main chain does not become higher under Inclusive, as the security guarantees
of main chain blocks apply to individual transactions as well (see the discussion
succeeding Algorithm 1). In particular, any acceptance policy employed by a
recipient of funds in a network following a non-inclusive protocol (see, e.g., [10,
12, 13]) can be safely carried out when Inclusive is implemented.

作为推论，
在Inclusive下一笔交易被排除在主链以外的可能性不会变得更高，
因为对主链区块的安全保证同样也适用于单笔交易（请参见算法1后面的讨论）。
【译注：
这里“算法1后面的讨论”具体应该是指第二章中命题（Proposition）1后面的一段话，
原文是
“An important property of this protocol is that once a transaction has been
approved by some main chain block B of G, it will remain in the approved set of
any extending block as long as B remains in G’s main chain...”，
译文是
“该协议的一个重要特性是，
一旦交易被G的某个主链区块B接受，
只要B保留在G的主链中，
该交易就会保留在任何B的扩展块的接受集中……”】
尤其是，
当Inclusive被实现的时候，
遵循非Inclusive协议
（例如，参见[10、12、13]）
的网络中的资金接受者所采用的任何接受策略都可以安全地被执行。

**Cost of Attacks** As mentioned at the beginning of this section, one may be
interested in measuring the cost of a double-spend attack rather than its success
probability. A potential drawback of including transactions from off-chain blocks
is that it mitigates the cost of a failed double-spend attack. Double spend attacks
consist typically of chains constructed by the attacker that are initially kept
secret. The construction of blocks requires computational resources. Under the
non-inclusive setup, when the attacker withdraws from the attack (usually after
failing to build blocks faster than the network), its blocks are discarded. In
contrast, under the Inclusive protocol, the attacker may still publish its secret
chain and gain some value from transactions contained inside.

**攻击成本**
如本节开头所述，
人们可能对双重花费攻击的成本而不是其成功率更感兴趣。
不剔除链下区块的交易的一个潜在缺点是它会减轻双花攻击的失败成本。
双花攻击通常来自攻击者构建的最初是秘密的链。
区块的构建需要计算资源。
在非Inclusive的设置中，
当攻击者撤回攻击时
（通常是在无法比网络更快地构建区块之后），
其区块将被丢弃。
相反，
在Inclusive协议下，
攻击者仍旧可以发布其秘密链并从其中所包含的交易获得一些收益。

However, the recipient of funds can cancel this effect by waiting longer before
accepting the payment. Indeed, if the attacker is forced to create long secret
chains, its blocks suffer some loss due to the lower reward implied by the function
γ(·).

但是，
资金的接收者可以在接受付款之前等待久一些从而抵消这种效果。
的确，
如果攻击者被迫制造很长的秘密链，
那么由于函数γ(·)暗含着更低的报酬，
其区块就会遭受一些损失。

To formalize this we provide first some definitions and notations. Denote by
G(t) the published developing block DAG at time t, and assume some main chain
block B<sub>tx</sub> confirms the transaction tx (that is, tx ∈
Inclusive-F(G(t),B<sub>tx</sub>, ∅)).  Let H(t) ⊆ G(t) be the set of blocks
from which B<sub>tx</sub> is reachable, and denote the main chain atop
B<sub>tx</sub> (including itself) by H<sub>main</sub>(t) ⊆ H(t). Let A(t) ⊆
G(t) \ H(t) be the set of blocks which satisfy post(·) ≻ Btx; these are blocks
which can be used by the attacker to reverse the transaction (even though the
attacker did not necessarily create all of them), and the requirement on their
post(·) block is to exclude from this set blocks earlier than B<sub>tx</sub>,
under the order of G (which do not affect the resolution of future conflicts).

为了对此进行形式化说明，
我们首先提供一些定义和符号。
定义G(t)为在时间t发布的正在增长的区块图，
并假设主链上的某个区块B<sub>tx</sub>确认了交易tx
（即 tx ∈ Inclusive-F(G(t),B<sub>tx</sub>,∅)）。
令H(t) ⊆ G(t)为可到达B<sub>tx</sub>的一组区块
【译注：即H(t)是B<sub>tx</sub>的后裔】，
并令H<sub>main</sub>(t) ⊆ H(t)表示B<sub>tx</sub>之上的主链
（包括B<sub>tx</sub>自身）。
设A(t) ⊆ G(t) \ H(t) 是满足post(·) ≻ B<sub>tx</sub>的区块集；
这些是攻击者可以利用来撤消该笔交易的区块
（即使它们并不一定全是由攻击者创建的），
并且对它们的post(·)有要求
【译注：即要求post(·) ≻ B<sub>tx</sub>】
是为了从这个集合
【译注：即A(t)】
中剔除G顺序下排序早于B<sub>tx</sub>的区块
（因为它们对将来解决冲突不会施加影响）。

---

#### 译注

下面这幅图是上面一段话定义的一堆符号的一个例子。

![](DefinitionsBeforeLemma3_Tony.jpeg)

---

Denote by val the expected reward from a block, under the Inclusive reward-scheme.
val is equal, in equilibrium, to the expected cost of creating a block.
We will simplify our analysis by assuming that val is constant. Finally, for convenience,
we analyze the case where the underlying chain selection rule (F) is
GHOST; the results apply to the longest-chain rule as well, after some slight
changes.

用val表示Inclusive奖励方案下一个区块的预期奖励。
val与区块创建的预期成本保持平衡。
我们将假定val为常数从而简化分析过程。
最后为方便起见，
我们假设底层的链选择规则（F）为GHOST;
分析结果在稍加调整后也适用于最长链规则。

---

#### 译注

关于奖励与成本之间的平衡关系，
可以参考《[区块链：技术驱动金融](https://www.amazon.cn/dp/B073QHSM7P/)》的第2.5节，
其中提到：

>我们预计矿工们会处在经济平衡点附近，
>意味着他们得到的奖励大致等于他们在硬件与电费上的花费。
>理由是如果一个矿工持续亏钱，
>他会停止挖矿。
>反之，
>如果硬件和电费固定的情况下，
>挖矿利润很高，那
更多的挖矿设备会加入网络。
>计算能力的增加会导致难度提高，
>每个矿工预期的回报便会降低。

equilibrium的意思是“平衡”而非“等价”。
收益和成本不一定相等，
但是从长期看它们会趋于大致相等。
下文引理3的证明中对受益的描述是“最多”，
而对成本的描述是“最少”也可以从侧面解释这个词的含义。

---

**Lemma 3.** Assume the attacker holds a fraction of at most q of the computational
power. If |H<sub>main</sub>(t)| = n, |A(t)| = m, and the attacker has created k secret
blocks, then the cost of a failed attack satisfies

**引理3：** 
假定攻击者最多持有算力的一部分，
比例为q。
如果|H<sub>main</sub>(t)| = n，
|A(t)| = m，
并且攻击者创建了k个秘密区块，
那么一次失败攻击的成本满足

<div align="center">
<img src="https://latex.codecogs.com/svg.latex?cost\geq\sum_{h=m&plus;1}^{m&plus;k}(1-\gamma(n&plus;2-h))&space;\cdot&space;val&space;\hfill&space;(3)" title="cost\geq\sum_{h=m+1}^{m+k}(1-\gamma(n+2-h)) \cdot val \hfill (3)" />
</div>

*Proof.* In the best case for the attacker, its blocks form a chain which is
built atop A(t). If A<sub>h</sub> is its h-th block (1 ≤ h ≤ k) then
pre(A<sub>h</sub>).height < B<sub>tx</sub>.height - 1 + m + h, or otherwise
A<sub>h</sub> necessarily references a block in H<sub>main</sub> as its main
parent (recall that a block’s ordered reference list is forced to agree with
F), and in particular it supports tx and does not participate in the attack.

*证明：*
在对攻击者最有利的情况下，
其区块形成一条构建在A(t)之上的链。
如果A<sub>h</sub>是其第h个块（1≤h≤k）
【译注：这里指A<sub>h</sub>是构建在A(t)之上的k个秘密区块中的第h个。
注意A<sub>h</sub>不在A(t)中，
因此它的高度可能会大于B<sub>tx</sub>】，
则pre(A<sub>h</sub>).height < B<sub>tx</sub>.height - 1 + m + h，
否则A<sub>h</sub>一定会引用H<sub>main</sub>中的一个区块作为其主父亲
（回想一下，某个区块的有序引用列表必须与F一致
【译注：这里指的是在一个区块的所有父亲中，
主链上的父亲必须排在其它父亲前面。
这句话解释了什么是“主父亲”。】），
那样它就会支持tx，
不参与攻击。

In addition, the attacker’s secret blocks are not published before the acceptance,
hence their post(·) block height is at least B<sub>tx</sub>.height + n. We conclude
that the discount parameter on Ah is at most

此外，
攻击者的秘密区块在tx被接受之前不会发布，
因此它们的post(·)区块的高度至少为B<sub>tx</sub>.height + n。
于是可以得出结论，
A<sub>h</sub>的折扣参数最多为

<div align="center">
<img src="https://latex.codecogs.com/svg.latex?\gamma((B_{tx}.height&plus;n)-(B_{tx}.height-1&plus;m&plus;h-1))," title="\gamma((B_{tx}.height+n)-(B_{tx}.height-1+m+h-1))," />
</div>

hence its cost is at least (1 − γ(n + 2 − m − h))·val. After a change of parameter
we arrive at (3).
<span>&#9744;</span>

因此，其成本至少
【译注：至少从长期来看，
也就是说当系统运行了很长时间，
奖励和成本大致相等的时候】
为(1-γ(n + 2 - m - h))·val。
更改参数后即可得出(3)。
<span>&#9744;</span>

# References

[10]. Nakamoto, S.: Bitcoin: A peer-to-peer electronic cash system (2008)

[12]. Rosenfeld, M.: Analysis of hashrate-based double spending. arXiv preprint
arXiv:1402.2009 (2014)

[13]. Sompolinsky, Y., Zohar, A.: Secure high-rate transaction processing in Bitcoin. In:
Financial Cryptography and Data Security. Springer (2015)
