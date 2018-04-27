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

**引理7.** *如果对于某个 $\hat{B} \in G$, $BLUE_k(G) \cap anticone(\hat{B}) = \emptyset$, 那么 $\forall B \in past(\hat{B}) \cap BLUE_k(G)$ 并且 $\forall C \notin past(\hat{B})$, $B \prec_G C$. 我们写作 $\hat{B} \in Hourglass(G)$.* （译注：Hourglass 在英文中的意思即是“沙漏”。根据该引理，$G$ 的蓝色区块一定在 $\hat{B}$ 的过去集、将来集和 $\hat{B}$ 自身当中。因此所有的蓝色区块组成的形状就像一个沙漏，而 $\hat{B}\$ 就是沙漏中间直径最小的两头相连的孔径。因此将 $\hat{B}$ 称为沙漏区块。注意如果 $\hat{B}$ 的反锥体中没有蓝色区块，那 $\hat{B}$ 自身一定是蓝色区块。这是由蓝色区块的选择算法即算法1保证的。该算法会在图中寻找一条从起始区块延伸至图末端的蓝色区块组成的链。如果 $\hat{B}$ 自己和其反锥体都是红色区块，那这条链就无法从起始区块眼神至末端了。）

In Figure 4 we provide an example of an Hourglass block. The proof of this lemma is straightforward from the operation of Algorithm 2:

在图4中我们给出了沙漏区块的一个例子。通过算法2的操作可以直接证明该引理：

![phantom_figure_4](https://user-images.githubusercontent.com/10098144/39358889-bb480462-4a52-11e8-9fa0-d02a2bcaa981.jpeg)
