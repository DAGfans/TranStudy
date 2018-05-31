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

*Proof of Claim 1.* Let $B \in anticone(\hat{B})$. If $\hat{B} \notin BLUE_k(G_s^{pub})$ we're done. Assume therefore that $\hat{B} \in BLUE_k(G_s^{pub})$. Any block that was published before time $u - D$ belongs to $past(\hat{B})$. Any block that was created after $u + D$ by an honest node belongs to $future(\hat{B})$. As honest nodes did not create blocks in the interim, blocks in $anticone(\hat{B})$ can only belong to the attacker. Now, since the attacher did not create new blocks while blocks in $LAST_k(past(\hat{B}))$ were created, all attacher blocks that were created before the interval $[u - T_1, u]$ and that do not belong to $past(\hat{B})$ (hence belong to $anticone(\hat{B})$) have all blocks in $LAST_k(past(\hat{B}))$ in their anticone; formally: $\forall C \in anticone(\hat{B}, G_u^{oracle}) : LAST_k(past(\hat{B})) \subseteq anticone(C, G_u^{oracle})$. Thus, if $\hat{B} \in BLUE_k(G_s^{pub})$, any block in $anticone(\hat{B})$ suffers an anticone that is larger than $k$ (it contains $LAST_k(past(\hat{B})) \cup {\hat{B}}$) and is therefore not in $BLUE_k(G_s^{pub})$. In particular, $B \notin BLUE_k(G_s^{pub})$.
