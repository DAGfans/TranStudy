>* **Source：** [https://eprint.iacr.org/2018/104.pdf](https://eprint.iacr.org/2018/104.pdf)  
>* **TranStudy：** [https://github.com/DAGfans/TranStudy/edit/master/Papers/PHANTOM/6-VARIANTS.md](https://github.com/DAGfans/TranStudy/edit/master/Papers/PHANTOM/6-VARIANTS.md)

# 6. VARIANT

# 6. 变体

In Section 2 we described the PHANTOM’s greedy algorithm to mark blocks as blue or red (Algorithm 1). In fact, similar algorithms can provide similar guarantees. We describe several such variants below and explain the intuition behind them. All these variants can be thought of as greedy approximations to the Maximum $k$-cluster SubDAG problem described in Section 1.

在第二部分中，我们描述了PHANTOM的贪婪算法，将块标记为蓝色或红色（算法1）。事实上，类似的算法能提供类似的保证。我们在下面描述几个这样的变体并解释它们背后的本质。所有这些变体都可以认为是第1节中描述的Maximum $k$-cluster SubDAG问题的贪婪近似。

*A. Choosing the maximizing tip*  
In our original version of the colouring procedure, we chose the tip which has the highest score (Algorithm 1, line 6). Instead, Algorithm 3 below chooses the tip for which the score of the (virtual block of the) current DAG would be highest:

*A.选择最大化末端*
在我们的着色程序的原始版本中，我们选择了得分最高的末端（算法 1，行 6）。相反，下面的算法3选择当前DAG（的虚拟块）的得分最高的末端：

> **Algorithm 3** Selection of a blue set

**Input:** G – a block DAG, k – the propagation parameter

**Output:** $BLUE_k(G)$ – the dense-set of G

1. **function** CALC-BLUE(G, k )

2. > **if** $G == {genesis}$ **then**

3. >> **return** {$genesis$}

4. > **for** $B \in tips(G)$ **do**

5. >> $BLUE_k(B) \leftarrow CALC-BLUE(past (B) , k)$

6. >> $S_B \leftarrow BLUE_k(B) \cup {B}$

7. >> **for** $C \in anticone(B)$ in some arbitrary order **do** 

8. >>> **if**$|anticone(C) \cap S_B| \leq k$ **then**

9. >>>> add C to $S_B$

10. >>**return** arg $max{|S_B|:B \in tips(G)}$ (and break ties arbitrarily)

*B. Adding blocks to the greedily chosen blue set*
Another variant over the previous algorithms is the procedure to mark more blocks as blue, after the best tip and its blue set were selected. Previously, we required that the candidate block admit an anticone of size at most k in the current set. Instead, we can require that its anticone be counted only inside the originally chosen blue set. Thus, yet another variant is to replace lines 8-9 with

*B.将块添加到贪婪选择出的蓝色集合*
在选择最优末端和蓝色集合之后，先前算法的另一个变体是将更多块标记为蓝色的过程。在此之前，我们要求候选块在当前集合中承认一个至多为k的反锥体。相反，我们可以要求只在最初选择的蓝色集合内计算反锥体。因此，另一个变式将代替上面的8-9行

8. >>> **if**$|anticone(C) \cap (BLUE_k(B) \cup {B})| \leq k$ **then**

9. >>>> add C to $S_B$

Observe that the resulting blue set may contain blocks which have an anticone larger than k within the set.

发现所得到的蓝色集合可以包含在该集合内具有大于k的反锥体的块。

Finally, another viable alternative is to further relax the condition (in lines 8-9) by counting the anticone of the candidate B only against the resulting chain of blue blocks, $Chn(past(B))\cup {B}$, where $B$ is the selected tip:

最终，另一个可行的选择是进一步放宽条件（在8-9行中），只计算与产生的蓝色块的链相反的候选人B的反锥体，$Chn(past(B))\cup {B}$，$B$就是被选择的末端：

8. >>> **if**$|anticone(C) \cap Chn(past(B))\cup {B}| \leq k$ **then**

9. >>>> add C to $S_B$

Compare this variant to Satoshi’s longest-chain rule, a rule that can be described as follows: each block in a chain $Chn$  increases its weight by 1, and we select the highest scoring chain. In contrast, in our last variant, each block whose gap from a chain $Chn$ is at most $k$ increases its weight by 1, and we select the highest scoring chain.

这个变体与中本聪的最长链比较，可以被描述为遵循以下规则：在$Chn$链中的每一个块以1增长自身的价值，我们选择得分最高的链。恰恰相反，在我们最后的一个变体中，每一个与$Chn$链差距至多为$k$的块以1增长自身的价值，我们选择得分最高的链。

*C. Iterative elimination of blocks*
We now introduce another variant based on an iterative method common in combinatorial optimization. The algorithm works as follows: Given a block DAG $G$, we iteratively eliminate from the blue set the block with largest blue anticone, and continue doing so until all blue blocks admit a blue anticone of size $k$ at most:

*C.块的迭代消除*
我们现在介绍另一种变体，基于一种常用于组合优化的迭代方法。算法描述如下：已知一个块DAG $G$，我们从蓝色集合中迭代消除有最大蓝色反锥体的块，并且一直重复直到所有蓝色块承认一个最大为$k$的蓝色反锥体：

> **Algorithm 4** Selection of a blue set

**Input:** G=(V,E) – a block DAG, k – the propagation parameter

**Output:** $BLUE_k(G)$ – the dense-set of G

1. **function** CALC-BLUE(G, k )

2. > $BLUE_k(G) \leftarrow V$

3. > **while** $\exists B \in BLUE_k(G)$  **with** $|anticone| \cap BLUE_k(G)|> k$ **do**

4. >> $C \leftarrow ard_B max{|anticone(B)\cap BLUE_k(G)|}$(with arbitrary tie-breaking)

5. >> remove $C$ from $BLUE_k(G)$

6. > **return** $BLUE_k(G)$

We conjecture that Algorithm 1 can be replaced with each of the greedy algorithms described in this section. To this end, suffice it to repeat the proof of Theorem 5 with respect to these variants. We suspect that the same proof technique, namely, the use of Hourglass, would prove useful.

我们猜想算法1可以用本节中描述的每个贪婪算法来代替。为此，只要重复定理5对于这些变体的证明就足够了。我们怀疑利用同样的证明技巧，即使是沙漏的使用，也将证明是有用的。

