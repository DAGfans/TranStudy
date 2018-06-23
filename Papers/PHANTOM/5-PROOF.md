>* **Source：** [https://eprint.iacr.org/2018/104.pdf](https://eprint.iacr.org/2018/104.pdf)  
>* **TranStudy：** [https://github.com/DAGfans/TranStudy/edit/master/Papers/PHANTOM/5-PROOF.md](https://github.com/DAGfans/TranStudy/edit/master/Papers/PHANTOM/5-PROOF.md)


# 5. PROOF

# 5. 证明

We now turn to prove that the order converges, and that an attacker (with less than 50%) is unable to cause reorgs. We restate the theorem from Section 3:

我们现在来证明顺序会收敛，以及攻击者（少于50%）无法导致重排序。我们将第3节的定理再复述一遍：

**Theorem 5**  (PHANTOM scales). *Given a block creation rate $\lambda > 0, \delta > 0$, and $D_{max} > 0$, if $D_{max}$ is equal to or greater than the network’s propagation delay diameter $D$, then the security threshold of PHANTOM, parameterized with $k(D_{max} , \delta)$, is at least $1/2 \cdot (1-\delta )$.*

**定理5** (PHANTOM 可扩容) *给定区块创建率 $\lambda > 0, \delta > 0$ 且 $D_{max} > 0$ ，如果 $D_{max}$ 大于或等于网络的传播延迟直径 $D$，则作为 $k(D_{max} , \delta)$ 参数的 PHANTOM 的安全阈值至少为 $1/2 \cdot (1-\delta )$。*

Our proof relies on the following lemma, which states that if some block B has the property that its anticone contains no blue blocks, then all blue blocks in its past precede all blocks outside its past. We call this the Hourglass property:

我们的证明依赖下面的引理。该引理说的是，若某个区块 B 的反锥体内没有蓝色区块，则其过去集中的所有蓝色区块在其过去集以外的蓝色区块之前。我们把这称为沙漏特性：

**Lemma 7.** *If for some $\hat{B} \in G$, $BLUE_k(G) \cap anticone(\hat{B}) = \emptyset$, then $\forall B \in past(\hat{B}) \cap BLUE_k(G)$ and $\forall C \notin past(\hat{B})$, $B \prec_G C$. We then write $\hat{B} \in Hourglass(G)$.*

**引理7：** *如果对于某个 $\hat{B} \in G$, $BLUE_k(G) \cap anticone(\hat{B}) = \emptyset$, 那么 $\forall B \in past(\hat{B}) \cap BLUE_k(G)$ 并且 $\forall C \notin past(\hat{B})$, $B \prec_G C$. 我们写作 $\hat{B} \in Hourglass(G)$.* （译注：Hourglass 在英文中的意思即是“沙漏”。根据该引理，$G$ 的蓝色区块一定在 $\hat{B}$ 的过去集、将来集和 $\hat{B}$ 自身当中。具体原因见下面引理7的证明部分。因此所有的蓝色区块组成的形状就像一个沙漏，而 $\hat{B}\$ 就是沙漏中间直径最小的两头相连的孔径。故将 $\hat{B}$ 称为沙漏区块。引理7换句话说，即沙漏区块的过去集里的蓝色区块在排序时一定在沙漏区块过去集以外的区块之前。）

![phantom_figure_4](https://user-images.githubusercontent.com/10098144/40277558-9f37438a-5c5b-11e8-851e-4a81bed989d3.png)

**Fig. 4:** An example of a DAG with an Hourglass block $K$. Here, the delay parameter is $k = 3$. As in Figure 3, the small circle near each block represents its score. The colouring of the DAG was done according to Algorithm 1. The greedily selected chain of blocks of highest score is marked with light-blue filling (note that this chain is not the longest one). Block $K$ has the property that all blue blocks are either in $K$’s past or in its future (in addition to $K$ itself). It is thus an Hourglass block.

**图4：** 含有一个沙漏区块 $K$ 的 DAG 的例子。在这里，延迟参数是 $k = 3$。和图3一样，每个区块旁边的小圆圈代表它的得分。我们依据算法1对 DAG 着色。由贪婪算法选出的最高得分的区块组成的链被标为浅蓝填充色（注意这条链并不是最长的链）。区块 $K$ 具有这样的性质：所有的蓝色区块要么在 $K$ 的过去集中，要么在其将来集中（除了 $K$ 自己）。因此它是一个沙漏区块。

译注：在图4里，区块 M 的得分是9；然而我们看到在 M 的过去集中却只有8个区块（即创世区块、B、C、D、F、I、J 和 K）被标注为蓝色。这并不是图有误。实际上当我们描述一个区块是否蓝色的时候，必须要先假设现在是站在哪个区块的视角来回答这个问题。回顾一下文章第2节中的步骤1和图3所描述的蓝色区块的选择算法。图4所标注的蓝色区块集记为 $BLUE_k(G)$，这等同于图中添加一个不存在的区块 V，然后让 M、R、U 作为 V 的父亲，最后从 V 的视角进行红蓝标注的结果。**每个区块的得分是在该区块刚发布连入图的时候，从该区块本身的视角进行红蓝标注之后，该区块过去集中蓝色区块的数量。文章对于这一点并没有描述得很清楚，这是译者在 DAGlabs 的 Slack 频道讨论后得知的结果。原话是：“Aviv & Yonatan 's algorithm is an iteration procedure, when you check $M$ you must stand on $M$'s viewpoint, that means $L$ is also blue on $M$'s viewpoint. Same for $H, N, Q, S, T, U$.”。**从图4的拓扑结构，我们可以推断，当 M 发布的时候，图中只有 L 和 K 两个末端区块（tip），E 的孩子和 K 的孩子都还没发布出来（注意区块发布的时候一定会连接图里面所有的末端区块）。这个时候，从 M 的角度看，L 是蓝色的，于是 M 的过去集中的蓝色区块就包括了创世区块、B、C、D、F、I、J、K 和 L 九个区块。因此 M 的得分为9。

In Figure 4 we provide an example of an Hourglass block. The proof of this lemma is straightforward from the operation of Algorithm 2:

在图4中我们给出了沙漏区块的一个例子。通过算法2的操作可以直接证明该引理：

*Proof of Lemma 7.* First note that if $BLUE_k(G) \cap anticone(\hat{B}) = \emptyset$, then $\hat{B} \in BLUE_k(G)$. Indeed, Algorithm 1 defines a chain, $Chn(G)$ (see Subsection 2.2). This chain necessarily intersects some block in anticone $anticone(\hat{B}) \cup \{\hat{B}\}$. And the intersection block must become blue itself, by lines (6-7). Thus, $BLUE_k(G) \cap anticone(\hat{B}) = \emptyset$ implies that $\hat{B} \in Chn(G)$ and in particular $\hat{B} \in BLUE_k(G)$.

*引理7的证明。* 首先注意到如果 $BLUE_k(G) \cap anticone(\hat{B}) = \emptyset$，则 $\hat{B} \in BLUE_k(G)$。确实，算法1定义了一条链，$Chn(G)$（见2.2节）。这条链一定会和 $anticone(\hat{B}) \cup \{\hat{B}\}$ 中的某个区块相交。根据算法的6-7行，相交的区块自身一定是蓝色。（译注：可以反过来理解。算法1在图中寻找一条从起始区块延伸至图末端的蓝色区块组成的链。如果 $\hat{B}$ 自己和其反锥体都是红色区块，那这条链就无法从起始区块眼神至末端了。）因此，$BLUE_k(G) \cap anticone(\hat{B}) = \emptyset$ 意味着 $\hat{B} \in Chn(G)$，尤其是 $\hat{B} \in BLUE_k(G)$。

Now, Algorithm 2 pushes a block into the queue only after it has pushed already all blocks in its past (lines 11 and 12). Therefore, $topo\_queue$ pops out blocks according to a topological order. Now, there are no blue blocks in $anticone(\hat{B}$, hence if $C$ is a blue block then it must belong to $future(\hat{B}) \cup \{\hat{B}\}$, in which case it must have been pushed to the queue after $\hat{B}$ was popped (unless $C = \hat{B}$). Thus, if $C$ is blue, any block in $past(\hat{B})$ was popped out before $C$, and added to the ordered list $L$ before it (line 8). In particular, $B \prec_G C$. On the other hand, if $C$ is not blue, it was necessarily pushed into the queue only by belonging to the past set of some blue block $B'$ (lines 9-11). $B'$ was popped out after $\hat{B}$ (unless $B' = \hat{B}$), from the same argument as in the previous case. Thus, $C$ was pushed into the queue after $\hat{B}$ was popped out, which in turn must have occurred after $B$ was popped out. In particular, in this case as well, $B \prec_G C$. $\hspace{5mm}\square$

现在，在算法2中，对任意一个区块来说，只有当该区块过去集中的所有区块都被压入队列之后，该区块才会被压入队列中（11和12行）。因此，$topo\_queue$ 会按照某种拓扑顺序弹出区块。因此，如果 $C$ 是蓝色区块，那么它一定属于 $future(\hat{B}) \cup \{\hat{B}\}$，于是它一定会在 $\hat{B}$ 被弹出之后才被压入队列中（除非 $C = \hat{B}$）。因此，如果 $C$ 是蓝色，那么 $past(\hat{B})$ 中的任意一个区块都会在 $C$ 之前被弹出，并在它之前被添加到有序列表 $L$ 中（第8行）。也就是说，$B \prec_G C$。另一方面，如果 $C$ 不是蓝色，它一定只能通过从属于某个蓝色区块 $B'$ 的过去集从而被压入队列中（第9-11行）。$B'$ 是在 $\hat{B}$ 之后被弹出的（除非 $B' = \hat{B}$），于是证明思路和前一种情况一样。因此， $C$ 被压入队列发生在 $\hat{B}$ 被弹出队列之后，而 $\hat{B}$ 被弹出又发生在 $B$ 被弹出之后。于是，这种情况下同样也有 $B \prec_G C$。 $\hspace{5mm}\square$

**Lemma 8.** *If $\hat{B}$ is an Hourglass block in a DAG $G$, then $G$ inherits the order from $\hat{B}$: $ord(G) \cap past(\hat{B}) = ord(past(\hat{B}))$.*

**引理8：** *如果 $\hat{B}$ 是 DAG $G$ 中的一个沙漏区块，那么 $G$ 继承来自 $\hat{B}$ 的顺序：$ord(G) \cap past(\hat{B}) = ord(past(\hat{B}))$。*

*Proof.* In the proof of the previous lemma we have shown that $\hat{B} \in Chn(G)$. By the recursive operation of Algorithm 1 (lines 6-7), this implies that $G$ inherits the colouring of $\hat{B}$ on its past: $BLUE_k(G) \cap past(\hat{B}) = BLUE_k(past(\hat{B}))$.

*证明：* 在上一个引理的证明中，我们已经展示了 $\hat{B} in Chn(G)$。算法1的递归操作（第6-7行）意味着 $G$ 继承了 $\hat{B}$ 的过去集的着色：$BLUE_k(G) \cap past(\hat{B}) = BLUE_k(past(\hat{B}))$。

Now, no block in $anticone(\hat{B})$ is pushed into $topo\_queue$ before $\hat{B}$ is popped out, because blocks get pushed in only if they are blue (and no such block exists in $anticone(\hat{B})$) or if a blue block in their future was pushed in-—and this too only happens after $\hat{B}$ is popped out. Since the queue respects the topology, this means that all blocks in $past(\hat{B})$ (which must be visited before $\hat{B}$) are popped out before any block outside $past(\hat{B})$.

现在，在 $\hat{B}$ 从 $topo\_queue$ 中被弹出之前，$anticone(\hat{B})$ 中没有区块被压入 $topo\_queue$。这是因为一个区块必须要满足以下两个条件之一才会被压入队列：要么该区块是蓝色（而这样的区块在 $anticone(\hat{B})$ 中并不存在）；要么该区块将来集中的某个蓝色区块被压入队列——这也只能发生在 $\hat{B}$ 被弹出之后。因为队列遵循拓扑，所以 $past(\hat{B})$ 中的所有区块（它们肯定都在 $\hat{B}$ 之前被访问过）都会在 $past(\hat{B})$ 以外的任意区块之前被弹出。

The fact that blocks in $anticone(\hat{B})$ are not pushed into the queue before all of $past(\hat{B})$ is popped out, implies further that the order in which Algorithm 2 pushes blocks in $past(\hat{B})$ into and out of $topo\_queue$ depends only on $past(\hat{B})$, and not on the DAG $G$. Hence, $ord(G) \cap past(\hat{B}) = ord(past(\hat{B})$. $\hspace{5mm}\square$

在 $past(\hat{B})$ 的所有区块被弹出之前，$anticone(\hat{B})$ 中的区块不会被压入队列。这进一步意味着算法2将 $past(\hat{B})$ 里的区块压入和弹出 $topo\_queue$ 的顺序仅取决于 $past(\hat{B})$，而不取决于 DAG $G$。因此，$ord(G) \cap past(\hat{B}) = ord(past(\hat{B})$。 $\hspace{5mm}\square$

The proof of Theorem 5 uses the occurrence of Hourglass blocks to secure all blocks in their past set.

定理5的证明利用沙漏区块来保证沙漏区块过去集里所有区块的安全性。

*Proof of Therom 5.* Let $\delta > 0$ and assume that $\alpha < 1/2 \cdot (1 - \delta)$. We need to prove that $\forall t_0 > 0$ and $B \in G_{t_0}^{pub}$: $\lim\limits_{t_1 \to \infty} Risk(B, t_1) = 0$, where $Risk(B, t_1) := \Pr(\exists s > t_1, \exists C \in G_s^{pub}: B \prec_{G_{t_1}^{pub}} C \wedge C \prec_{G_s^{pub}} B)$, and assuming a fraction of at most $1/2 - \delta$ can deviate arbitrarily from the mining pool.

*定理5的证明：* 假设 $\delta > 0$ and assume that $\alpha < 1/2 \cdot (1 - \delta)$。我们需要证明对于 $\forall t_0 > 0$ 和 $B \in G_{t_0}^{pub}$： $\lim\limits_{t_1 \to \infty} Risk(B, t_1) = 0$，其中 $Risk(B, t_1) := \Pr(\exists s > t_1, \exists C \in G_s^{pub}: B \prec_{G_{t_1}^{pub}} C \wedge C \prec_{G_s^{pub}} B)$，并且假设不遵守挖矿协议的节点占比最多只有 $1/2 - \delta$。

Fix $t_0$ and $B$, and let $\tau(t_0)$ be the first time after $t_0$ where an honest block $\hat{B}$ was published such that $\hat{B}$ is an Hourglass block and will forever remain so:

将 $t_0$ 和 $B$ 固定，并且假设 $t_0$ 之后符合下面条件的一个诚实区块 $\hat{B}$ 被发布——该区块是 $t_0$ 之后首个被发布的诚实的沙漏区块，并且之后也将一直保持为沙漏区块。我们将该区块的发布时间记为 $\tau(t_0)$：

$$
\tau(t_0) := min \{ u \geq t_0 : \exists \hat{B} \in G_u^{pub} \text{ such that } \hat{B} \in \cap_{r \geq s}Hourglass(G_r^{pub}) \} \hspace{1cm} (2)
$$

$\tau(t_0)$ is a random variable. By Lemma 9 it has finite expectation. In particular, $\lim\limits_{t_1 \to \infty} \Pr(\tau(t_0) > t_1) = 0$ (e.g., by Markov’s Inequality).

$\tau(t_0)$ 是一个随机变量。根据定理9，它的期望是有限的。特别是，$\lim\limits_{t_1 \to \infty} \Pr(\tau(t_0) > t_1) = 0$ （比如，根据马尔可夫不等式）。

Assume that such a $\tau(t_0)$ arrives, i.e., that a block $\hat{B}$ is created and remains permanently an Hourglass block. Then, by Lemma 8, the order between all blocks in $past(\hat{B})$ never changes, and in particular the order between $B$ and any other block $C$ never changes. Thus, $Risk(B, \tau(t_0)) = 0$. Therefore, $\lim\limits_{t_1 \to \infty} \Pr(\tau(t_0) > t_1) = 0$ implies $\lim\limits_{t_1 \to \infty} Risk(B, t_1) = 0$. $\hspace{5mm}\square$

假设这样的一个 $\tau(t_0)$ 出现了，也就是有一个区块 $\hat{B}$ 被创建并且永远保持为一个沙漏区块。接着，根据引理8，$past(\hat{B})$ 里面所有区块之前的顺序都永不改变，并且尤其是 $B$ 和其它任意一个区块 $C$ 之间的顺序都永不改变。于是，$Risk(B, \tau(t_0) = 0$。因此，$\lim\limits_{t_1 \to \infty} \Pr(\tau(t_0) > t_1) = 0$ 意味着 $\lim\limits_{t_1 \to \infty} Risk(B, t_1) = 0$。 $\hspace{5mm}\square$

**Lemma 9.** *Let $t_0 \geq 0$. The expected waiting time for $\tau(t_0)$ (defined in (2)) is finite, and moreover is upper bounded by a constant that does not depend on $t_0$.*

**引理9：** *假设 $t_0 \geq 0$。$\tau(t_0)$ 的期望等待时间（在（2）中定义）是有限的，并且其上限是一个不依赖 $t_0$ 的常数。*

*Proof.* Let $\varepsilon(t_0)$ denote the event defined by the following conditions:

*证明：* 将 $\varepsilon(t_0)$ 记为满足下列条件的事件：

1\) Some block $\hat{B}$ was created at some time $u > t_0$ by an honest node (i.e., $\hat{B} \in G_u^{pub}$) and apart from $\hat{B}$ no other block was created in the time interval $[u - D, u + D]$. ^13

1\) 某个区块 $\hat{B}$ 在某个时刻 $u > t_0$ 被一个诚实节点所创建（即 $\hat{B} \in G_u^{pub}$）并且除了 $\hat{B}$ 以外在时间区间 $[u - D, u + D]$ 以内没有其它区块被创建。 ^13

^13 We cannot assume, however, that no block was published during this interval, because the attacker might decide to publish during this interval blocks that he created earlier.

^13 然而，我们不能假设在此区间内没有区块被发布，因为攻击者可能会决定在此区间内发布他早前创建的区块。

2\) For some $T_1$, the $k$ last blocks in $BLUE_k(past(\hat{B}))$, denoted $LAST_k(past(\hat{B}))$, was created in the time interval $[u - T_1, u]$, and dishonest miners created no blocks in this time interval.

2\) 对某个时间长度 $T_1$ 来说，$BLUE_k(past(\hat{B}))$ 的最后 $k$ 个区块，记为 $LAST_k(past(\hat{B}))$，是在时间区间 $[u - T_1], u]$ 内被创建的，并且在此时间区间内作弊矿工没有创建区块。

3\) The score of $\hat{B}$'s chain is forever higher than the score of any chain that does not pass through $\hat{B}$: $\forall s \geq u, \forall C_1, C_2 \in tips(G_s^{pub}): score(C_1) \geq score(C_2) \Longrightarrow \hat{B} \in Chn(past(C_1))$.

3\) $\hat{B}$ 的链的得分永远比任意一条不穿过 $\hat{B}$ 的链的得分高：$\forall s \geq u, \forall C_1, C_2 \in tips(G_s^{pub}): score(C_1) \geq score(C_2) \Longrightarrow \hat{B} \in Chn(past(C_1))$。（译注：感觉这里的数学符号表达并不严谨，因为有可能 $Chn(past(C_1))$ 和 $Chn(past(C_2))$ 都不包含 $\hat{B}$。但我并不确定。可能需要详细理解下文断言3的证明。）

Claim 2 and 4 together imply the required result.

由断言2和4可共同推导出所需的结果（即引理9）。

The following claim states that the first two conditions in the above definition guarantee that as long as $\hat{B}$ is blue no block in its anticone is blue:

下面的断言表示，上面定义中的前两个条件保证了只要 $\hat{B}$ 是蓝色的，它的反锥体中就没有区块是蓝色的：

**Claim 1.** *Assume that for some block $\hat{B}$ the first two conditions in the definition of $\varepsilon(t_0)$ hold true. Then, as long as $\hat{B}$ is blue, all blue blocks have $\hat{B}$ in their past: $\forall s \geq u, \forall B \in anticone(\hat{B}) : B \notin BLUE_k(G_s^{pub}) \lor \hat{B} \notin BLUE_k(G_s^{pub})$.*

**断言1：** *假设某个区块 $\hat{B}$ 满足 $\varepsilon(t_0)$ 定义中的前两个条件。那么，只要 $\hat{B}$ 是蓝色的，所有蓝色区块的过去集都会包含 $\hat{B}$：$\forall s \geq u, \forall B \in anticone(\hat{B}) : B \notin BLUE_k(G_s^{pub}) \lor \hat{B} \notin BLUE_k(G_s^{pub})$。*

（译注：断言1换句话说，也就是All blocks created by the attacker before u-T_1 are either in \past(\hat{B}) or in the anticone of all k+1 blocks in Last (including \hat{B}) itself）

*Proof of Claim 1.* Let $B \in anticone(\hat{B})$. If $\hat{B} \notin BLUE_k(G_s^{pub})$ we're done. Assume therefore that $\hat{B} \in BLUE_k(G_s^{pub})$. Any block that was published before time $u - D$ belongs to $past(\hat{B})$. Any block that was created after $u + D$ by an honest node belongs to $future(\hat{B})$. As honest nodes did not create blocks in the interim, blocks in $anticone(\hat{B})$ can only belong to the attacker. 
Now, since the attacher did not create new blocks while blocks in $LAST_k(past(\hat{B}))$ were created, all attacher blocks that were created before the interval $[u - T_1, u]$ and that do not belong to $past(\hat{B})$ (hence belong to $anticone(\hat{B})$) have all blocks in $LAST_k(past(\hat{B}))$ in their anticone; 
formally: $\forall C \in anticone(\hat{B}, G_u^{oracle}) : LAST_k(past(\hat{B})) \subseteq anticone(C, G_u^{oracle})$. 
Thus, if $\hat{B} \in BLUE_k(G_s^{pub})$, any block in $anticone(\hat{B})$ suffers an anticone that is larger than $k$ (it contains $LAST_k(past(\hat{B})) \cup {\hat{B}}$) and is therefore not in $BLUE_k(G_s^{pub})$. In particular, $B \notin BLUE_k(G_s^{pub})$.

*断言 1 证明：* 假设$B \in anticone(\hat{B})$，如果已知$\hat{B} \notin BLUE_k(G_s^{pub})$。因此假设$\hat{B} \in BLUE_k(G_s^{pub})$。任何在$u - D$之前发布的块属于$past(\hat{B})$。任何在$u + D$之后由诚实节点创造的块属于$future(\hat{B})$。由于诚实的节点在此期间并未创建块，因此$ anticone（\ hat {B}）$中的块只能属于攻击者。
现在，因为在创建$LAST_k(past(\hat{B}))$中的块时，附加器没有创建新块，所以在时间间隔$[u - T_1, u]$之前所有附加块被创建并且不属于$past(\hat{B})$的（因此属于$anticone(\hat{B})$）所有附加块，拥有所有在它们的反锥体中的$LAST_k(past(\hat{B}))$中的所有块；
形式上：$\forall C \in anticone(\hat{B}, G_u^{oracle}) : LAST_k(past(\hat{B})) \subseteq anticone(C, G_u^{oracle})$。
因此，如果$\hat{B} \in BLUE_k(G_s^{pub})$，任何在$anticone(\hat{B})$中的块容许一个大于$k$的反锥体(它由$LAST_k(past(\hat{B})) \cup {\hat{B}}$组成)，因此不在$BLUE_k(G_s^{pub})$中。尤其是，$B \notin BLUE_k(G_s^{pub})$。


**Claim 2.** *The waiting time for the event $\varepsilon (t_0)$ upper bounds $\tau(t_0)$.*

**断言 2:** *事件$\varepsilon (t_0)$的等待时间的上界为$\tau(t_0)$*

*Proof of Claim 2.* The previous claim implies that a block $\widehat{B}$ that satisfies the first two conditions is an Hourglass block in $G_s^{pub}$. The third condition implies that $\widehat{B}$ forever remains in the chain, and in particular forever remains blue. It is thus an Hourglass block in all DAGs $G_s^{pub}(s \geq u)$.

*断言 2 证明：* 之前的断言表明一个满足前两个条件的块$\widehat{B}$是在$G_s^{pub}$中的一个Hourglass块。第三个条件表明$\widehat{B}$永远在链中，尤其是永远是蓝色的。因此一个Hourglas块在所有的DAGs $G_s^{pub}(s \geq u)$中。

**Claim 3.** *If the first two conditions in the definition of $\varepsilon (t_0)$ hold true then the third one holds true with a positive probability.*

**断言3:** *如果$\varepsilon (t_0)$定义中的前两个条件成立，则第三个条件以正概率成立.*

*Proof of Claim 3.* Part I: We begin by assuming that the attacker did not publish any block in 􏰇the time interval $[0,u-D_max)$; formally, we assume that $\forall C \in  past(\widehat{B}):C\in honest$.

*断言 3 证明：* 第一部分：我们首先假设攻击者在时间间隔$[0,u-D_max)$内没有发布任何块；形式上，我们假设为$\forall C \in  past(\widehat{B}):C\in honest$。

Let us compare the score of the honest chain to that of any chain that excludes $\widehat(B)$􏰇. This gap is captured by the following definition: For a time $r > 0$, define

让我们比较诚实链和任何排除$\widehat(B)$􏰇的链的得分。以下定义可以看出这一差距：对于时间$r > 0$，定义

  $X^{1}_r:=\max_{B:\widehat{B} \notin Chn(B)}\{score(Chn(B))\}$
  $X^{2}_r:=\max_{B:\widehat{B} \in Chn(B)}\{score(Chn(B))\}$
  $X_r:=X^{1}_r-X^{2}_r$

Let us focus first on the evolution of the process $X_r$ between time 0 and time $u−T_1$. We refer to the lead $X_u−T_1$ ，that the attacker obtained at the end of this stage as “the premining gap”; see [8].

让我们首先关注时间0到$u−T_1$之间的过程$X_r$的演变。我们指的是领先的$X_u−T_1$，攻击者在本阶段结束时获得作为“预排空隙”的$X_u−T_1$；参考【8】。

Let $B^{r}_1$ be the argmax of $X^{1}_r$, and let $C^{r}_1$ be the latest block in $G^{pub}_r \cap BLUE_k(past(B^{r}_1))$, namely, the latest honest block which is blue in the attacker chain. 

让$B^{r}_1$作为$X^{1}_r$argmax函数的变量，让$C^{r}_1$成为$G^{pub}_r \cap BLUE_k(past(B^{r}_1))$中最新的块，即，在攻击者链上的蓝色的最新的诚实块。

Recall that for now we are assuming that all attacker blocks that were premined were kept secret until after time $u−D_max$. Observe that at most k blocks that were created by the attacker before $time(􏰀C^{r}_1)$􏰁 can be in $BLUE_k(past(B^{r}_1))$ and can contribute to the score of the attacker’s chain. 

回想一下，现在我们假设所有被预设的攻击者块都保密，直到时间$u−D_max$之后。观察到在$time(􏰀C^{r}_1)$􏰁之前由攻击者创建的至多k个块可以在$BLUE_k(pst(B^{r}_1))$中，并且可以贡献得分给攻击者链条。

Thus, between $time(􏰀C^{r}_1)$􏰁 and time $u−T_1-i.e.$, during the premining phase – the score of the attacker chain grows only via the contribution of attacker blocks, and therefore at a rate of $\alpha \cdot \beta$ at most.^{14}

因此，在时间$time(􏰀C^{r}_1)$􏰁和$u−T_1-i.e.$期间，在预测阶段 - 攻击者链的得分只能通过攻击者块的贡献增加，因此最多以$\alpha \cdot \beta$的速率。

^{14}Notice that our analysis does not assume that the attacker creates its blocks in a single chain. We only claim that the attacker’s highest scoring chain grows at a rate of $\alpha \cdot \beta$ at most, because every attacker block can increase the attacker’s highest scoring chain by 1 at most. Creating a single chain is indeed the optimal attack on the attacker side.

^{14}注意我们的分析没有假设攻击者在单链上创建它的块。我们仅仅声明攻击者最高得分链最多以$\alpha \cdot \beta$的速率增长，因为每一个攻击者块最多能以1的速率增加攻击者最高得分链。创建单链确实是以攻击者角度的最佳攻击。

Now, by the choice of $K(D_{max},\delta )$ (defined in 1), the probability of an arbitrary honest block $B$ having too large of an honest anticone is small:^{15}

现在，通过$K(D_{max},\delta )$（在定义 1中）的选择，任意诚实块$B$具有太大的诚实反锥体的概率是小的：^{15}

$\Pr_{B\sim arbitrary\ honest\ block\ in\  G^{pub}_t}\(|\overline{anticone_h}(B,G^{pub}_t)| > k(D_{max},\delta ))$

This is because block creation follows a Poisson process, and because honest blocks that were created $D-{max}$ seconds before (after) $B$ belong to its past (future), hence are not in its anticone. Since any block that is blue in the honest chain contributes to its score, at any time interval, the score of the public chain grows at a rate of $(1-\delta)\cdot (1-\alpha )\cdot \lambda$ at least. And at most k honest blocks created in the premining stage contributed to the score of the attack chain.

这是因为块创建遵循泊松过程,并且因为被创建于$B$之前（之后）$D-{max}$秒的诚实块属于它的过去（未来），因此不在它的反锥体中。因为诚实链上任何蓝色的块都有助于其得分，在任何时间间隔，公共链的得分至少以$(1-\delta)\cdot (1-\alpha )\cdot \lambda$的速率增长。最多k个诚实块创建在预期阶段对攻击链的得分做出贡献的链上。

*Part III: From Part I and Part II* we conclude that the random process $X_r-k$ is upper bounded by the premining race analyzed in [8]. Therein it was shown that the probability distribution over $X_{u - T_1} - k$ (the process’s state at the end of the premining stage) is dominated by the
stationary probability distribution $\pi$ of a reflecting random walk over the non-negative integers with a bias of $\frac{(1-\delta)\cdot (1-\alpha)}{1-\delta\cdot (1-\alpha )}$ towards negative infinity. The stationary distribution exists because $\alpha\leq 1/2\cdot (1-\delta) < (1-\alpha)\cdot (1-\delta)$.^{16}

*第三部分: 来自 第一部分 和 第二部分*
我们得出这样的结论：在[8]中分析的预期竞争是随机过程$X_r-k$的上限。其中显示超过$X_{u - T_1} - k$的概率分布（在预期阶段最后的进程阶段）占有主要地位，通过一个反向随机游走带有$\frac{(1-\delta)\cdot (1-\alpha)}{1-\delta\cdot (1-\alpha )}$到负无穷偏差的非负整数的平稳概率分布$\pi$。平稳概率分布存在因为$\alpha\leq 1/2\cdot (1-\delta) < (1-\alpha)\cdot (1-\delta)$。^{16}

In particular, there is a positive probability that at time $u - T_1$ the premining gap, $X_u-T_1$ , was less than k, so that $X_u−T_1 - k < 0$.^{17} Between times $u - T_1$ and $u$ the honest network contributed additional $k + 1$ to the score of the public chain, namely, blocks $LAST_k(past(\widehat{B}))\cup \{\widehat{B}\}$. During the same time the score of any secret chain(s) did not increase at all, per the second condition in the definition of $\varepsilon (t_0)$. Thus, $X_u = X_u−T_1 - (k + 1)$. And by the first assumption, $X_u + D_{max} = X_u$. All in all, with some positive probability, $X_u + D_{max} < 0$.

特别是，在时间$u - T_1$预先存在的差距有一个正的概率，$X_u-T_1$，少于k，因此$X_u−T_1 - k < 0$。^{17}在时间$u - T_1$和$u$之间，诚实网络为公共链得分贡献了额外的$k + 1$，即，块$LAST_k(past(\widehat{B}))\cup \{\widehat{B}\}$。在相同的时间内，任何秘密链的得分不再增加，根据定义中的第二个条件$\varepsilon (t_0)$。因此，$X_u = X_u−T_1 - (k + 1)$。根据第一个假设，$X_u + D_{max} = X_u$。总而言之，在一些正概率下，$X_u + D_{max} < 0$。

*Part IV:* Let us turn to look at the evolution of $(X_r)_r \leq u+D_{max}$ at the second stage. Assume that $X_u + D_{max} < 0$.

*第五部分：* 让我们来看看第二阶段$(X_r)_r \leq u+D_{max}$的演化。假设$X_u + D_{max} < 0$。

In Claim 1 we saw that for any $r$ such that $\widehat{B} \in BLUE_k(G^{pub}_r)$􏰄, all blocks in $\widehat{B}$’s anticone are red in $G^pub_r$. This implies that, as long as $\widehat{B}$ is blue in the public DAG, only attacker blocks contribute to the score of the attacker’s chain: $\widehat{B} \in BLUE_k(G^{pub}_r) \Rightarrow \forall B \in anticone(\widehat{B}): BLUE_k(past(B))\setminus  future(\widehat{B}) = \not{0}$
􏰇(indeed, note that all honest blocks created after time $u + D_{max}$ belong to $future(\widehat{B})􏰇$). Consequently, the attacker’s best chain grows at a rate of
$\alpha \cdot \beta$ at most, as this interval ($[u+D_{max},\infty )$) as well, as long as $\widehat{B} \in BLUE_k(G^pub_r)$􏰄. We have already seen that at any time interval the honest chain’s score grows at a rate of $(1−\delta )·(1−\alpha )·\lambda$  at least.

在断言1中，我们我们看到，对于任何$r$，例如$\widehat{B} \in BLUE_k(G^{pub}_r)$􏰄,所有在$\widehat{B}$的反锥体中的区块在$G^pub_r$中是红色的。这意味着，只要$\widehat{B}$在公共DAG中是蓝色的，仅仅攻击者块贡献攻击者链的得分：ibute to the score of the attacker’s chain: $\widehat{B} \in BLUE_k(G^{pub}_r) \Rightarrow \forall B \in anticone(\widehat{B}): BLUE_k(past(B))\setminus  future(\widehat{B}) = \not{0}$（实际上，注意在时间$u + D_{max}$之后创建的所有诚实区块属于$future(\widehat{B})􏰇$）。所以，攻击者的最优链最多以$\alpha \cdot \beta$的速率增长,和这个时间间隔($[u+D_{max},\infty )$)一样，只要$\widehat{B} \in BLUE_k(G^pub_r)$􏰄。我们已经看到在任何时间间隔内，诚实块的得分至少以$(1−\delta )·(1−\alpha )·\lambda$的速率增长。

^{15}Below, $\overline{anticoneh}(B, G)$ denotes all blocks in $anticone(B, G)$ created by honest nodes.

^{15}以下，$\overline{anticoneh}(B, G)$表示所有在$anticone(B, G)$中的块由诚实结点创建。

^{16}Note that we have multiplied $(1−\alpha)$ by $(1−\delta)$ to account for the rare events where there was a burst of block creation events and where consequently some honest blocks did not contribute to the score of the public chain. Note on the other hand that blocks created by honest nodes too do not contribute to the attacker chain’s score, even if they are red in $G_{pub}^r$. This is because we argued that at most $k$ blocks that were created by the attacker before time􏰀 ($C^r_1$)􏰁 can be blue in $BLUE_k (past(B^r_1))$, and this is regardless of $C^r_1$’s status within $G_{pub}^r$ (we merely used the fact that $C^r_1$ was created by an honest node, we didn’t use the assumption that $C^r_1$ is blue in the honest chain). 

^{16}注意我们已经把$(1−\alpha)$和$(1−\delta)$相乘来解释罕见的事件，如有一些块创建事件突然发生，因此一些诚实块没有贡献公共链的得分。注意，另一方面，由诚实结点创建的块也没有贡献攻击者链的得分，即使它们在$G_{pub}^r$是红色的。因为我们认为由攻击者在时间（$C^r_1$）之前创建的至多$k$块在$BLUE_k  (past(B^r_1))$中是蓝色的，并且这与$G_{pub}^r$中的$C^r_1$的状态无关（我们仅仅运用事实$C^r_1$在诚实链中是蓝色，我们不使用假设$C^r_1$在诚实链中是蓝色的）。

^{17}In fact, this probability is decreasing logarithmically as $k$ increases.

^{17}事实上，这个概率随着$k$增加而以对数递减。

Thus, starting at time $u+D_{max}$, the race between the honest chain’s score and the attacker chain’s score can be modeled as a random walk over all integers with a bias of $\frac{(1-\delta )(1-\alpha )}{1-\delta\cdot (1-\alpha )}$ towards negative infinity.

因此，在时间$u+D_{max}$时开始，在诚实链和攻击者链得分之间的竞争能建模为，一个随机遍历所有伴有$\frac{(1-\delta )(1-\alpha )}{1-\delta\cdot (1-\alpha )}$到负无穷偏差的整数。

Since the random walk begins at the negative location $X_u+D_{max}$, this supposedly implies that with a positive probability the walk will never return to the origin: $Pr (\forall r\geq u+ D_{max} : X_r < 0) > 0$. This in turn implies that $\widehat{B}$􏰇 will forever remain a blue block (and a chain block for that matter). However, to complete the analysis correctly, some careful attention is required:

因为随机遍历起始于负位置$X_u+D_{max}$，据推测，这意味着以正概率遍历将永远不会回到原点：$Pr (\forall r\geq u+ D_{max} : X_r < 0) > 0$。这轮流表明了$\widehat{B}$将永远保持蓝色（和在这种情况下仍是链块）。然而，为了正确地完成分析，需要特别注意：

*Part V:* First, we must account for the fact that not all honest nodes observe all honest blocks immediately, i.e., that $G^v_r$ might be a proper subset of $G^{pub}_r$. This might give the attacker an advantage, as he can create blocks, reveal them immediately to honest nodes, and these blocks do not compete with honest blocks unseen yet by these nodes. This advantage can be accounted for by assuming that the race begins only after the attacker was given $D_{max}$ additional seconds to create blocks while the honest network sat idle; see [8]. This fact does not change our general conclusion that $Pr (\forall r\geq u+ D_{max} : X_r < 0) > 0$, e.g., because there is a positive probability that the attacker did not create any block during these $D_{max}$ additional seconds.

*第五部分：*首先，我们必须解释，不是所有的诚实节点都可以立即发现诚实块，i.e.，$G^v_r$可能是$G^{pub}_r$的一个合适的子集。这可能给攻击者带来优势，因为他可以创建区块，立即把他们展示给诚实块，这些块不能和仍没有被这些节点发现的块竞争。这个优势能被解释，通过假设竞争仅在攻击者被给予$D_{max}$的额外秒数用来创建块而诚实网在闲置状态之后才开始：参考[8]。这个事实没有改变我们的一般结论 $Pr(\forall r\geq u+ D_{max} : X_r < 0) > 0$，e.g.，因为攻击者在这些$D_{max}$额外秒数的时间内没有创建任何块。

Secondly, observe that the honest chain grows according to a random process that is not necessarily Poisson. Fortunately, a result from [9] shows that nevertheless we can treat the block race as if the honest score grows according to a Poisson process, and that this assumption can only increase the value of $X_r$; it is thus a worst case analysis.

第二，根据不一定是泊松分布的随机分布观察诚实链增长。幸运地，来自[9]中的结果表明虽然我们对待块竞争就好像诚实链根据泊松分布增长一样，并且这个假设仅可以增加$X_r$的价值；因此这是一个最坏的情况分析。

*Part VI:* Finally, we alleviate our assumption that the attacker published no block during the premining phase $[0,u−D_{max})$. Instead, consider any attacker block $C$ that belongs to $past(􏰃\widegat{B})$􏰄. If $C\in BLUE_k(past(BB))$ then $C$ contributed to the score of the honest chain which passes through $\widehat{B}$􏰇 and maybe to the attacker chain as well, so its existence does not change the above analysis. Similarly, if $C\notin BLUE_k(past(BB))$, the fact that it was published has no consequence whatsoever on the score of the honest chain, and so we can ignore it and apply the same analysis as if it weren’t published.

*第五部分：*最终，我们调整了我们的假设，即攻击者在预测阶段$[0,u−D_{max})$时没有发布任何块。代替的是，考虑任何属于$past(􏰃\widegat{B})$的攻击者块$C$。如果$C\in BLUE_k(past(BB))$，$C$贡献得分给通过$\widehat{B}$的诚实链，可能也贡献给攻击者链，因此它的存在不能改变以上的分析。相似地，如果$C\notin BLUE_k(past(BB))$，它发布的事实对诚实链的得分没有任何影响，因此我们能忽略它并且采用相同的分析就好像它没有被发布。

**Claim 4.** *The waiting time for the event $\varepsilon (t_0)$ is finite. Moreover, it is upper bounded by a constant that does not depend on $t_0$.*

**断言 4.** *事件$\varepsilon (t_0)$的等待时间是有限的。而且，它的上限由一个不依赖于$t_0$的常数决定。*

*Proof of Claim 4.* Let $B$ be an arbitrary honest block created in $time(B)$. The probability that no other block was created in the interval $[time (B)-D_{max}, time (B)+D_{max}]$ is given by $e^{-2\cdot D_{max}\cdot \lambda }$. An arbitrary block $B$ satisfies the first condition in the definition of $\varepsilon (t_0)$ with a positive probability.

*断言4 证明:* 令让$B$成为在$time(B)$中创建的任意诚实块。在时间间隔$[time (B)-D_{max}, time (B)+D_{max}]$内没有创建其他块的概率由$e^{-2\cdot D_{max}\cdot \lambda }$决定。随机块$B$在正概率下满足$\varepsilon (t_0)$的第一个条件。

Given an arbitrary block $B$ satisfying the above condition, let $T_1$ be the creation time of the earliest block in $LAST_k(past (B))$. The probability that the attacker did not create any block in the interval $[u-T_1, u-D_{max}]$ is given by $((1-\alpha )\cdot (1-\delta ))^k$. In particular, given the first condition, the second one is satisfied with a positive probability.

给定任意块$B$满足上述条件,让$T_1$是在$LAST_k(past (B))$中最早被创建块的创建时间。在时间间隔$[u-T_1, u-D_{max}]$内攻击者没有创建其他块的概率由$((1-\alpha )\cdot (1-\delta ))^k$决定。特别是，给定第一个条件，第二个条件满足正概率。

Importantly, for any two blocks $B_1$ and $B_2$ created after $t_0$ and that satisfy $|time(B_1)-time(B_2)| > 4\cdot  D_{max}$, the satisfaction of the first condition with respect to $B_1$ is independent from its satisfaction with respect to $B_2$. Consequently, the expected waiting time for the occurrence of a block $\widehat{B}$􏰇 which satisfies the first two conditions in the definition of $\varepsilon (t_0)$ is finite (see, for instance, Chapter 10.11 in [10]). Moreover, while the precise expected time $\mathbb{E}[Hourglass(t_0)]$ may theoretically depend on t0, the above argument shows that $\mathbb{E}[Hourglass(t_0)]< const + 4 \cdot d$, where const does not depend on $t_0$.
This completes the proof of Claim 4 and of Lemma 9.

重要的是，对于任意两个块$B_1$和$B_2$在$t_0$后创建，并且满足$|time(B_1)-time(B_2)| > 4\cdot  D_{max}$，满足关于$B_1$的第一个条件与满足$B_2$是独立的。因此，满足$\varepsilon (t_0)$的定义中前两个条件的块出现的预期等待时间是有限的（参见，例如，章节10.11 [10]）。此外，尽管精确的预期时间$\mathbb{E}[Hourglass(t_0)]$在理论上可能取决于t0，但上述论证表明$\mathbb{E}[Hourglass(t_0)]< const + 4 \cdot d$，其中const不依赖于$t_0$。
这完成了断言4和引理9的证明。

Theorem 5 guarantees that the probability of reorg with respect to a given block $B$ diminishes: $Risk(B, t_1)\rightarrow 0$. However, it does not guarantee anything about the convergence rate, i.e., the waiting time for a $t_1$ that satisfies $Risk(B, t_1) <\varepsilon $, for some $\varepsilon > 0$.^{18} Following the analysis in the proof of Claim 4, the waiting time for an Hourglass block can be upper bounded by a constant in the order of magnitude of $O(e^{C\cdot D_{max}\cdot \lambda })$􏰁, for some $C > 0$; and after such a block is created,the analysis implies that $Risk(B,t_1)$ converges to 0 at an exponential rate, due to the random walk dynamic.

定理5保证相对于给定区块$B$的重组概率减少：$Risk(B, t_1)\rightarrow 0$。然而，它并不能保证收敛速率，i.e.，对于$t_1$满足the waiting time for a $t_1$ that satisfies $Risk(B, t_1) <\varepsilon $的等待时间，对于$\varepsilon > 0$。^{18}。在断言4的证明分析中，沙漏块的等待时间的上界由一个在$O(e^{C\cdot D_{max}\cdot \lambda })$􏰁数量级的常数限制，对于$C > 0$；在这样一个块被创建之后，分析表明，由于随机遍历是动态的，$Risk(B,t_1)$以指数速率收敛到0。

However, the analysis used in the proof is not tight. It relies on rather infrequent events which guarantee convergence in a straightforward way, namely, on the occurrence of Hourglass blocks. In practice the network is likely to converge much faster. Moreover, it can be shown that when there is no active attack, the convergence time is faster by orders-of-magnitude.

然而，在证明中使用的分析并不严密。它依靠相当罕见的事件来保证以简单的方式进行收敛，即，出现沙漏块。实际上，网络可能会更快地收敛。此外，可以看出，当没有主动攻击时，收敛比数量级要快。


