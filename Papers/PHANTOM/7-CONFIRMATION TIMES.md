>* **Source：** [https://eprint.iacr.org/2018/104.pdf](https://eprint.iacr.org/2018/104.pdf)  
>* **TranStudy：** [https://github.com/DAGfans/TranStudy/edit/master/Papers/PHANTOM/7-CONFIRMATION TIMES.md](https://github.com/DAGfans/TranStudy/edit/master/Papers/PHANTOM/7-CONFIRMATION TIMES.md)
# 7. CONFIRMATION TIMES

# 7. 确认时间

As discussed in Section 5, the convergence rate of $Risk(B,t)$ is slow, at least theoretically and under certain circumstances. Recall that the function $Risk(B,t)$ measures the probability that a certain block that did not precede $B$ at time $t$ will later come to precede it. Recall further that throughput this work we used arbitrary topological orderings (over blue blocks). In light if this, it would be interesting to seek for an ordering rule (over blue blocks) that would converge faster. We suspect this is not a trivial task, and leave its full investigation to future work.

如第5节所述，$Risk(B,t)$的收敛速度很慢，至少在理论上和某些情况下。回想一下，函数$Risk(B,t)$计算在时间$t$时没有出现在$B$之前位置确定的块将在之后出现在它之前位置的可能性。进一步回忆一下，我们使用任意拓扑排序（在蓝色块之上）处理吞吐量。有鉴于此，寻求一种能够更快收敛的排序规则（通过蓝色块）会很有趣。我们认为这不是一项微不足道的任务，并且对未来的工作留下充分的调查。

The primary factor to the fact that **PHANTOM** cannot guarantee fast confirmation times is that membership in the blue set takes time to finalize. The waiting time for such finalization can be further increased if an attacker manages to balance the decision between $B \in BLUE_k(G)$ and $B \notin BLUE_k(G)$. Observe however that if a certain transaction $tx \in B$ admits no conflicts in $anticone(B)$, then $tx$ can be accepted even before the decision regarding $B$ is finalized.

**PHANTOM**无法保证快速确认时间这一事实的主要原因是，蓝色集合中的成员资格需要时间才能最终确定。如果攻击者在$B \in BLUE_k(G)$和$B \notin BLUE_k(G)$中设法平衡决定，那么等待时间会进一步增加。但是请注意，如果某个交易$tx \in B$承认在$anticone(B)$中没有冲突，甚至$tx$在关于$B$的决定已经完成之前就被接受。

*A. Combining SPECTRE and PHANTOM*

*A. 结合SPECTRE和PHANTOM*

SPECTRE is a DAG based protocol that can support large transaction throughput (similarly to *PHANTOM*) and very fast confirmations. SPECTRE does not output a linear order over blocks. Rather, every block $B$ admits a vote regarding the pairwise ordering of any two blocks $C$ and $D$, and the output is the majority vote regarding each pair. In SPECTRE, cycles of the sort “$B$ precedes $C$, $C$ precedes $D$, $D$ precedes $B$” may form.

SPECTER是一种基于DAG的协议，可以支持较大的交易吞吐量（与*PHANTOM*相似）并非常快速的确认。SPECTER不输出块的线性顺序。相反，每个块$B$承认关于任何两个块$C$和$D$的成对排序的投票,并且输出是关于每一对的最多数票。在SPECTRE中，“$B$在$C$之前，$C$在$D$之前，$D$在$B$之前”的环可能形成。

Furthermore, if an active balancing attack is taking place, for some pairs of blocks $(B, C)$ the SPECTRE relation might not become robust (in which case $Risk(B, t) \rightarrow 0$ is not guaranteed). Instead, a published block $B$ is guaranteed to robustly precede any block $C$ that was published later than $B$, unless $C$ was published shortly after $B$. We call this property *Weak Liveness*. In the context of the Payments application, this fact can only harm a user that signed and published two conflicting payments at approximately the same time.^{19}

此外，如果发生主动平衡攻击，对于某些块$(B, C)$，SPECTER关系可能变得不稳固（在这种情况下$Risk(B, t) \rightarrow 0$不能被保证）。相反，已发布的块$B$被保证可以稳固地位于比$B$晚发布的任何块$C$之前的位置，除非$C$在$B$后不久发布。我们称这个属性为"虚弱的”活性"。在支付申请的环境下，这一事实只会对几乎同时签署和发布两笔冲突付款的用户造成伤害。^{19}

Since *PHANTOM* does guarantee (strong) Liveness and a liner ordering, it is interesting to inquire whether we can enjoy the best of both worlds. We provide below a partial answer to this question.

因为*PHANTOM*能保证（强壮的）“活性”和线性排序，询问我们是否能享受两全其美的好处很有意思。我们在下面提供这个问题的部分答案。

Consider the following procedure: Given a blockDAG $G$,

> 1) mark blocks as blue or red according to PHANTOM’s colouring procedure

> 2) run SPECTRE on the subDAG $BLUE_k(G)$; this determines the pairwise ordering between any two $blue$ blocks $B$ and $C$

> 3) for any blue block $B$ and red block $C$, if $C \in past(B)$ then determine that $C$ precedes $B$, otherwise determine that $B$ precedes $C$

> 4) decide the pairwise ordering of any two red blocks in some arbitrary way that respects the topology ($B \in past(C) \Rightarrow B$ precedes $C$)

考虑以下过程：已知DAG块$G$，

> 1) 根据PHANTOM的着色程序将块标记为蓝色或红色

> 2) 在subDAG $BLUE_k(G)$上运行SPECTRE；
这决定了任何两个$蓝色$块$B$和$C$之间成对排序

> 3) 对于任何蓝色块$B$和红色块$C$，如果$C \in past(B)$则确定$C$在$B$之前，否则确定$B$在$C$之前
 
> 4) 以某种符合拓扑的任意方式决定任意两个红色块的成对排序($B \in past(C) \Rightarrow B$在$C$之前)

Intuitively, we run SPECTRE on the set of blue blocks (as determined by PHANTOM), and
complemented the pairwise ordering in a way that both penalizes red blocks and respects the topology. We argue that this protocol enjoys SPECTRE’s fast confirmation times and at the same time inherits the (regular) Liveness property of PHANTOM. To see the latter, observe that a Hourglass block would have a similar effect in SPECTRE——all blocks in its future will vote according to its vote. In particular, the pairwise relation of all previous blocks becomes as robust as the Hourglass block.

直观地说，我们在蓝色块集合上运行SPECTER（由PHANTOM决定），并以一种既惩罚红色方块并符合拓扑结构的方法来补充成对排序。我们认为，该协议享有SPECTER的快速确认时间的性质，同时继承了PHANTOM的（常规）“活性”的属性。要观察后者，观察沙漏块在SPECTER中会有类似的效果——未来所有区块将根据其投票数进行投票。特别是，所有先前块的成对关系变得和沙漏块一样稳固。

Note that this incremental improvement over vanilla SPECTRE is only possible because we allowed the protocol to assume something on the network’s topology, namely, that the communication delay diameter is upper bounded by $D_{max}$.

需要注意的是，对于常规SPECTER的这种增量式改进是可能的，因为我们允许协议在网络的拓扑结构上假设某些条件，即，通信延迟直径上限为$D_{max}$。


*B. Summary*

In summary, it is possible to achieve both fast confirmation times and Liveness by combining PHANTOM and SPECTRE. It is of yet unclear whether we can further achieve a linear ordering without compromising the fast confirmation times. Hopefully, we will provide answers to these questions in future work.

*B. 总结*
总之，通过组合PHANTOM和SPECTER可以实现快速确认时间和活性。目前还不清楚我们是否可以在不影响快速确认时间的情况下进一步实现线性排序。希望我们能在未来的工作中为这些问题提供答案。

^{19}In contrast, usually any user can engage with a smart contract and introduce conflicts inputs. Thus, Weak Liveness might potentially harm the usability of SPECTRE to the Smart Contracts application.

^{19}相比之下，通常任何用户都可以参与智能合约并引入冲突输入。因此，弱活性可能会损害SPECTER对智能合同应用程序的可用性。

##REFERENCES
##参考文献

[1] Ittay Eyal, Adem Efe Gencer, Emin Gu ̈n Sirer, and Robbert Van Renesse. Bitcoin-ng: A scalable blockchain protocol. In 13th USENIX Symposium on Networked Systems Design and Implementation (NSDI 16), pages 45–59, 2016.
[2] Michael R Garey and David S Johnson. Computers and intractability, volume 29. wh freeman New York, 2002.
[3] Aggelos Kiayias and Giorgos Panagiotakos. On trees, chains and fast transactions in the blockchain. Cryptology
ePrint Archive, Report 2016/545, 2016.
[4] Eleftherios Kokoris Kogias, Philipp Jovanovic, Nicolas Gailly, Ismail Khoffi, Linus Gasser, and Bryan Ford.
Enhancing bitcoin security and performance with strong consistency via collective signing. In 25th USENIX
Security Symposium (USENIX Security 16), pages 279–296. USENIX Association, 2016.
[5] Yoad Lewenberg, Yonatan Sompolinsky, and Aviv Zohar. Inclusive block chain protocols. In International
Conference on Financial Cryptography and Data Security, pages 528–547. Springer, 2015.
[6] Rafael Pass and Elaine Shi. Hybrid consensus: Efficient consensus in the permissionless model, 2016.
[7] Joseph Poon and Thaddeus Dryja. The bitcoin lightning network: Scalable off-chain instant payments. Technical
Report (draft), 2015.
[8] Yonatan Sompolinsky, Yoad Lewenberg, and Aviv Zohar. Spectre: A fast and scalable cryptocurrency protocol.
IACR Cryptology ePrint Archive, 2016:1159, 2016.
[9] Yonatan Sompolinsky and Aviv Zohar. Secure high-rate transaction processing in bitcoin. In International
Conference on Financial Cryptography and Data Security, pages 507–527. Springer, 2015.
[10] David Williams. Probability with martingales. Cambridge university press, 1991.
