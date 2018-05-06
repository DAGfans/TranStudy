>* **Source：** [https://eprint.iacr.org/2018/104.pdf](https://eprint.iacr.org/2018/104.pdf)  
>* **TranStudy：** [https://github.com/DAGfans/TranStudy/edit/master/Papers/PHANTOM/5-PROOF.md](https://github.com/DAGfans/TranStudy/edit/master/Papers/PHANTOM/5-PROOF.md)


# 5. PROOF

# 5. 证明

We now turn to prove that the order converges, and that an attacker (with less than 50%) is unable to cause reorgs. We restate the theorem from Section 3:

我们现在来证明顺序会收敛，以及攻击者（少于50%）无法导致重排序。我们将第3节的定理再复述一遍：

**Theorem 5**  (PHANTOM scales). *Given a block creation rate $\lambda > 0, \delta > 0$, and $D_{max} > 0$, if $D_{max}$ is equal to or greater than the network’s propagation delay diameter $D$, then the security threshold of PHANTOM, parameterized with $k(D_{max} , \delta)$, is at least $1/2 \cdot (1-\delta )$.*

**定理5** (PHANTOM 可扩展) *给定区块创建率 $\lambda > 0, \delta > 0$ 且 $D_{max} > 0$ ，如果 $D_{max}$ 大于或等于网络的传播延迟直径 $D$，则作为 $k(D_{max} , \delta)$ 参数的 PHANTOM 的安全阈值至少为 $1/2 \cdot (1-\delta )$。*

Our proof relies on the following lemma, which states that if some block B has the property that its anticone contains no blue blocks, then all blue blocks in its past precede all blocks outside its past. We call this the Hourglass property:

我们的证明依赖下面的引理。该引理说的是，若某个区块 B 的反锥体内没有蓝色区块，则其过去集中的所有蓝色区块在其过去集以外的蓝色区块之前。我们把这称为沙漏特性：

**Lemma 7.** *If for some $\hat{B} \in G$, $BLUE_k(G) \cap anticone(\hat{B}) = \emptyset$, then $\forall B \in past(\hat{B}) \cap BLUE_k(G)$ and $\forall C \notin past(\hat{B})$, $B \prec_G C$. We then write $\hat{B} \in Hourglass(G)$.*

**引理7：** *如果对于某个 $\hat{B} \in G$, $BLUE_k(G) \cap anticone(\hat{B}) = \emptyset$, 那么 $\forall B \in past(\hat{B}) \cap BLUE_k(G)$ 并且 $\forall C \notin past(\hat{B})$, $B \prec_G C$. 我们写作 $\hat{B} \in Hourglass(G)$.* （译注：Hourglass 在英文中的意思即是“沙漏”。根据该引理，$G$ 的蓝色区块一定在 $\hat{B}$ 的过去集、将来集和 $\hat{B}$ 自身当中。具体原因见下面引理7的证明部分。因此所有的蓝色区块组成的形状就像一个沙漏，而 $\hat{B}\$ 就是沙漏中间直径最小的两头相连的孔径。故将 $\hat{B}$ 称为沙漏区块。引理7换句话说，即沙漏区块的过去集里的蓝色区块在排序时一定在沙漏区块过去集以外的区块之前。）

![phantom_figure_4](https://user-images.githubusercontent.com/10098144/39358889-bb480462-4a52-11e8-9fa0-d02a2bcaa981.jpeg)

**Fig. 4:** An example of a DAG with an Hourglass block $K$. Here, the delay parameter is $k = 3$. As in Figure 3, the small circle near each block represents its score. The colouring of the DAG was done according to Algorithm 1. The greedily selected chain of blocks of highest score is marked with light-blue filling (note that this chain is not the longest one). Block $K$ has the property that all blue blocks are either in $K$’s past or in its future (in addition to $K$ itself). It is thus an Hourglass block.

**图4：** 含有一个沙漏区块 $K$ 的 DAG 的例子。在这里，延迟参数是 $k = 3$。和图3一样，每个区块旁边的小圆圈代表它的得分。我们依据算法1对 DAG 着色。由贪婪算法选出的最高得分的区块组成的链被标为浅蓝填充色（注意这条链并不是最长的链）。区块 $K$ 具有这样的性质：所有的蓝色区块要么在 $K$ 的过去集中，要么在其将来集中（除了 $K$ 自己）。因此它是一个沙漏区块。

In Figure 4 we provide an example of an Hourglass block. The proof of this lemma is straightforward from the operation of Algorithm 2:

在图4中我们给出了沙漏区块的一个例子。通过算法2的操作可以直接证明该引理：

*Proof of Lemma 7.* First note that if $BLUE_k(G) \cap anticone(\hat{B}) = \emptyset$, then $\hat{B} \in BLUE_k(G)$. Indeed, Algorithm 1 defines a chain, $Chn(G)$ (see Subsection 2.2). This chain necessarily intersects some block in anticone $anticone(\hat{B}) \cup \{\hat{B}\}$. And the intersection block must become blue itself, by lines (6-7). Thus, $BLUE_k(G) \cap anticone(\hat{B}) = \emptyset$ implies that $\hat{B} \in Chn(G)$ and in particular $\hat{B} \in BLUE_k(G)$.

*引理7的证明。* 首先注意到如果 $BLUE_k(G) \cap anticone(\hat{B}) = \emptyset$，则 $\hat{B} \in BLUE_k(G)$。确实，算法1定义了一条链，$Chn(G)$（见2.2节）。这条链一定会和 $anticone(\hat{B}) \cup \{\hat{B}\}$ 中的某个区块相交。根据算法的6-7行，相交的区块自身一定是蓝色。（译注：可以反过来理解。算法1在图中寻找一条从起始区块延伸至图末端的蓝色区块组成的链。如果 $\hat{B}$ 自己和其反锥体都是红色区块，那这条链就无法从起始区块眼神至末端了。）因此，$BLUE_k(G) \cap anticone(\hat{B}) = \emptyset$ 意味着 $\hat{B} \in Chn(G)$，尤其是 $\hat{B} \in BLUE_k(G)$。

Now, Algorithm 2 pushes a block into the queue only after it has pushed already all blocks in its past (lines 11 and 12). Therefore, $topo\_queue$ pops out blocks according to a topological order. Now, there are no blue blocks in $anticone(\hat{B}$, hence if $C$ is a blue block then it must belong to $future(\hat{B}) \cup \{\hat{B}\}$, in which case it must have been pushed to the queue after $\hat{B}$ was popped (unless $C = \hat{B}$). Thus, if $C$ is blue, any block in $past(\hat{B})$ was popped out before $C$, and added to the ordered list $L$ before it (line 8). In particular, $B \prec_G C$. On the other hand, if $C$ is not blue, it was necessarily pushed into the queue only by belonging to the past set of some blue block $B'$ (lines 9-11). $B'$ was popped out after $\hat{B}$ (unless $B' = \hat{B}$), from the same argument as in the previous case. Thus, $C$ was pushed into the queue after $\hat{B}$ was popped out, which in turn must have occurred after $B$ was popped out. In particular, in this case as well, $B \prec_G C$. $\square$

现在，在算法2中，对任意一个区块来说，只有当该区块过去集中的所有区块都被压入队列之后，该区块才会被压入队列中（11和12行）。因此，$topo\_queue$ 会按照某种拓扑顺序弹出区块。因此，如果 $C$ 是蓝色区块，那么它一定属于 $future(\hat{B}) \cup \{\hat{B}\}$，于是它一定会在 $\hat{B}$ 被弹出之后才被压入队列中（除非 $C = \hat{B}$）。因此，如果 $C$ 是蓝色，那么 $past(\hat{B})$ 中的任意一个区块都会在 $C$ 之前被弹出，并在它之前被添加到有序列表 $L$ 中（第8行）。也就是说，$B \prec_G C$。另一方面，如果 $C$ 不是蓝色，它一定只能通过从属于某个蓝色区块 $B'$ 的过去集从而被压入队列中（第9-11行）。$B'$ 是在 $\hat{B}$ 之后被弹出的（除非 $B' = \hat{B}$），于是证明思路和前一种情况一样。因此， $C$ 被压入队列发生在 $\hat{B}$ 被弹出队列之后，而 $\hat{B}$ 被弹出又发生在 $B$ 被弹出之后。于是，这种情况下同样也有 $B \prec_G C$。 $\square$

**Lemma 8.** *If $\hat{B}$ is an Hourglass block in a DAG $G$, then $G$ inherits the order from $\hat{B}$: $ord(G) \cap past(\hat{B}) = ord(past(\hat{B}))$.*

**引理8：** *如果 $\hat{B}$ 是 DAG $G$ 中的一个沙漏区块，那么 $G$ 继承来自 $\hat{B}$ 的顺序：$ord(G) \cap past(\hat{B}) = ord(past(\hat{B}))$。*

*Proof.* In the proof of the previous lemma we have shown that $\hat{B} \in Chn(G)$. By the recursive operation of Algorithm 1 (lines 6-7), this implies that $G$ inherits the colouring of $\hat{B}$ on its past: $BLUE_k(G) \cap past(\hat{B}) = BLUE_k(past(\hat{B}))$.

*证明：* 在上一个引理的证明中，我们已经展示了 $\hat{B} in Chn(G)$。算法1的递归操作（第6-7行）意味着 $G$ 继承了 $\hat{B}$ 的过去集的着色：$BLUE_k(G) \cap past(\hat{B}) = BLUE_k(past(\hat{B}))$。

Now, no block in $anticone(\hat{B})$ is pushed into $topo\_queue$ before $\hat{B}$ is popped out, because blocks get pushed in only if they are blue (and no such block exists in $anticone(\hat{B})$) or if a blue block in their future was pushed in-—and this too only happens after $\hat{B}$ is popped out. Since the queue respects the topology, this means that all blocks in $past(\hat{B})$ (which must be visited before $\hat{B}$) are popped out before any block outside $past(\hat{B})$.

现在，在 $\hat{B}$ 从 $topo\_queue$ 中被弹出之前，$anticone(\hat{B})$ 中没有区块被压入 $topo\_queue$。这是因为一个区块必须要满足以下两个条件之一才会被压入队列：要么该区块是蓝色（而这样的区块在 $anticone(\hat{B})$ 中并不存在）；要么该区块将来集中的某个蓝色区块被压入队列——这也只能发生在 $\hat{B}$ 被弹出之后。因为队列遵循拓扑，所以 $past(\hat{B})$ 中的所有区块（它们肯定都在 $\hat{B}$ 之前被访问过）都会在 $past(\hat{B})$ 以外的任意区块之前被弹出。

The fact that blocks in $anticone(\hat{B})$ are not pushed into the queue before all of $past(\hat{B})$ is popped out, implies further that the order in which Algorithm 2 pushes blocks in $past(\hat{B})$ into and out of $topo\_queue$ depends only on $past(\hat{B})$, and not on the DAG $G$. Hence, $ord(G) \cap past(\hat{B}) = ord(past(\hat{B})$. $\square$

在 $past(\hat{B})$ 的所有区块被弹出之前，$anticone(\hat{B})$ 中的区块不会被压入队列。这进一步意味着算法2将 $past(\hat{B})$ 里的区块压入和弹出 $topo\_queue$ 的顺序仅取决于 $past(\hat{B})$，而不取决于 DAG $G$。因此，$ord(G) \cap past(\hat{B}) = ord(past(\hat{B})$。 $\square$
