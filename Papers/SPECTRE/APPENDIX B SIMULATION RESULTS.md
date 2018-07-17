# APPENDIX B SIMULATION RESULTS
# 附录B 模拟结果

We implemented the SPECTRE protocol in Python along with an event-driven simulator of network dynamics.
For each experiment we generated an Erd˝os-R´enyi random network topology with 20 nodes. 
Each node forms 5 outgoing links, in expectation. 
The delay on each link was uniformly distributed and later scaled linearly so that the diameter of the graph is D (for the given D). 
Every point represents the average outcome over at least 500 experiments.

我们在Python中实现了SPECTRE协议以及一个事件驱动的网络动态模拟器。
对于每个实验，我们生成了具有20个节点的Erd˝os-R'enyi随机网络拓扑。
预期每个节点形成5个对外的连接。
每个连接上的延迟均匀分布，然后线性缩放，使得图的直径为D（对于给定的D）。
每个点代表至少500次实验的平均结果。


The main beneﬁt of SPECTRE is fast transaction conﬁrmation. 
The asymptotic waiting times derived from our formal analysis are in $O(\frac{ln(1/\varepsilon )}{\lambda (1-2\alpha )}+\frac{D}{1-2\alpha })$. 
In order to measure the actual waiting times, we utilized the online acceptance policy derived by Alg. 7. 
Accordingly, we stress that the merchant needs to wait additional D seconds in order to verify that no double-spend has been released in the past D seconds, as explained at the end of Appendix C. 18

SPECTRE的主要好处是快速的交易确认。
通过我们的形式化分析得到渐进等待时间在 $O(\frac{ln(1/\varepsilon )}{\lambda (1-2\alpha )}+\frac{D}{1-2\alpha })$ 以内。
为了测量实际的等待时间，我们采用了算法7中生成的在线接受策略。
因此，我们强调商家需要额外等待D秒才能确认在过去的D秒内没有发生双花，如附录C末尾所述。

How does the delay diameter affect acceptance times? 
Given that block creation rate is high, most of the waiting time for acceptance is dominated by the block propagation delay. 
Fig. 5 depicts the transaction acceptance times of SPECTRE, for various values of the delay diameter D, and for different security thresholds. 
Note that, unlike the Nakamoto Consensus, D affects the acceptance time of transactions but not their security.

延迟直径是怎样影响接受时间的？ 
鉴于出块率高，大多数的接受等待时间是由区块传播时间决定的。 
图5描绘了SPECTRE的交易接受时间, 针对多种延迟直径D, 以及不同的安全阈值。 
请注意，与中本聪共识不同，D影响交易的接受时间，但不影响其安全性。

<img width="418" alt="2018-07-17 2 09 12" src="https://user-images.githubusercontent.com/39436379/42799609-0a53e564-89cb-11e8-844c-ab0ee3d6b289.png">

Fig. 5: The average time for a transaction to enter RobustTxO, assuming there’s no visible double-spend, for $\lambda = 10$ blocks per second and $\alpha = 0.25$.

图5：交易进入RobustTxO状态的平均时间，假设没有可见的双花，$\lambda = 10$块每秒，$\alpha = 0.25$。

How does the block creation rate affect acceptance times? 
Fig. 6 depicts the acceptance times for various values of the block creation rate $\lambda$, under a constant delay $d = 5$ seconds. 
The graph reafﬁrms the role of $\lambda$ in our asymptotic bound: accelerating the block creation process allows for faster acceptance times. 
For comparison, Bitcoin’s block creation rate of 1/600 implies waiting times that are orders of magnitudes higher (not plotted).

块创建率如何影响接受时间？ 
图6描绘了在延迟恒定为$d = 5$秒时,  多种出块率$\lambda$对应的接受时间。 
图表重现了$\lambda$在我们的渐近界中的作用：加速块创建过程允许更快的接受时间。 相比之下，比特币的块创建率为1/600意味着等待时间更高（未绘制）。



Fig. 6: The average time for a transaction to enter RobustTxO, assuming there’s no visible double-spend, for d = 5 seconds and $\alpha = 0.25$.

Can an attacker delay acceptance? We now turn to demonstrate the effect of censorship attacks in which some dishonest nodes publish blocks that do not reference other miners’ blocks. Recall that the Weak Liveness property of SPECTRE (Proposition 3) guarantees fast acceptance of transactions that are not visibly double-spent, even in the presence of a censorship attack. However, such an attack still causes some delay in transaction acceptance, but this delay is minor for small attackers. In Fig. 7 we quantify this effect, by comparing the acceptance times in “peace days” to those under an active censorship attack. The parameters here are d = 5 seconds, $\lambda = 10$ blocks per second, and $\varepsilon = 0.01$. The results display a modest effect of the attack, and they show that in order to delay transaction acceptance by more than 5 to 10 seconds an attacker must possess a signiﬁcant share of the computational power in the network.

<img width="448" alt="2018-07-17 2 10 03" src="https://user-images.githubusercontent.com/39436379/42799635-24ed2a7a-89cb-11e8-9764-8aedf5b02364.png">

Fig. 7: The average time for a transaction to enter RobustTxO, assuming there’s no visible double-spend, for $d = 5$ seconds, $\lambda = 10$ blocks per second, and $\varepsilon = 0.01$, in the presence and in the absence of a censorship attack.

How does $\varepsilon$ decrease for various sizes of the attacker? Once an honest node $\varepsilon$-accepts a transaction, there’s still a small risk ($\varepsilon$) that it would eventually be rejected. We show that the probability of this event vanishes quickly, even for an extremely capable attacker (e.g., with $\alpha = 0.4$ of the hashrate). This is illustrated in Fig. 8, assuming $d = 5$ seconds and $\lambda = 10$ blocks per second (notice that the y-axis is in log scale).

How tight is our security analysis? The analysis on which Alg. 3 relies makes several worst-case assumptions in order to bound the probability of a successful attack, e.g., that the attacker can broadcast blocks to and receive blocks from all nodes without any delay (see Appendix E, mainly Lemmas 14 and 20). Accordingly, the analysis is not tight, and in reality attacks are in fact less likely to succeed. In Fig. 9, we depict the comparison between the analytical bound and two different empirical simulations. In these simulations we explicitly generate blocks for the attacker and simulate the optimal double-spending attack. We repeat the experiment 10,000 times for each point in the graph, and measure the empirical success rate. The simulations assume two types of attackers: a worst-case attacker that is able to transmit and receive blocks with no delays, and a more realistic attacker that is connected to other nodes with typical delays. We compared the fraction of successful attacks under these setups to the analytical risk calculated by SPECTRE’s policy (Alg. 7).

The results show that the risk considered by SPECTRE’s RiskTxAccept indeed upper bounds the actual risk, and that transactions are even safer than we guarantee formally.

<img width="433" alt="2018-07-17 2 26 21" src="https://user-images.githubusercontent.com/39436379/42800213-70a62c30-89cd-11e8-9743-bc9ad5438d83.png">

Fig. 8: The probability of a successful double-spending attack, as a function of the waiting time before acceptance, under $d = 5$ seconds and $\lambda = 10$ blocks per second, for $\alpha$ = 0.1, 0.25, and 0.4. The probability here is the result of the calculation performed by Alg.3.

<img width="424" alt="2018-07-17 2 27 17" src="https://user-images.githubusercontent.com/39436379/42800227-890b1560-89cd-11e8-842e-ca4eef626419.png">

Fig. 9: The analytical vs. empirical probabilities of a successful double-spending attack, as a function of the waiting time before acceptance, under $d = 5$ seconds, $\lambda = 10$, and $\alpha = 0.25$.
