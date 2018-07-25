> Source : https://blog.daglabs.com/confirmation-times-in-spectre-7f68fec0d997

> TranStudy : https://github.com/DAGfans/TranStudy/blob/master/Blogs/Confirmation%20Times%20in%20SPECTRE.md

# Confirmation Times in SPECTRE
# SPECTRE中的交易确认时间

Since it solves the orphan rate problem, SPECTRE can operate at very high block rates, resulting in transaction confirmation times in mere seconds (see previous blog post).   
Unlike Bitcoin, SPECTRE’s core protocol is agnostic to the network propagation delay;   
some upper bound on the propagation delay must be assumed, but this assumption is made locally by each node when determining transaction safety.  

因为SPECTRE解决了孤块率问题，因此SPECTRE可以以非常高出块率运行，从而在几秒钟内完成交易确认（参见上一篇博客文章）。  
与比特币不同，SPECTRE的核心协议与网络传播延迟无关；  
必须假设传播延迟有某个上限，但是这个假设是在确定交易安全时由每个节点在本地进行的情况下作出的。

Nodes that misestimate the propagation delay will hurt their own performance alone.   
This upper bound assumption can be adjusted locally as network conditions change, which allows SPECTRE to be responsive¹ to network conditions and achieve optimally fast confirmation times during healthy periods.   
In contrast, Bitcoin’s measure of ten-minute block times is carved in stone.
  
错误估计传播延迟的节点会降低自己的性能。  
当网络条件发生变化时，可以在本地调整此上限假设，这使得SPECTRE能够适应¹网络状况并在网络状况良好期间实现最佳的快速确认时间。  
相比之下，比特币的十分钟出块时间的参数是固定不变的。

## Reorgs in Bitcoin and blockDAGs

## 比特币和blockDAG中的重组

In Bitcoin, a node’s observed main chain will undergo a reorganization if the node discovers a new chain that excludes some blocks from its current main chain and is longer than its current main chain.   
The node will switch over to this new chain, and the excluded blocks become orphans.   
Slight reorganizations of the chain, if they occur, may be caused by network asynchronicity,   
whereas large reorganizations are likely to be caused by attackers with sufficient mining power to overtake the main chain, or by some catastrophic split in the network.

在比特币中，如果节点发现一条新链从当前主链中排除了某些块,并且新链比节点的当前主链长，则被该节点观察的主链将进行重组。  
节点将切换到此新链，被排除的块将成为孤块。  
链的轻微重组（如果发生的话）可能是由网络异步性引起的，  
而大型重组很可能是由挖掘能力足以超越主链的攻击者引起的，或者是由网络中的某些灾难性分裂引起的。

![1_jkfjdohkk7xciwh3qle6pg](https://user-images.githubusercontent.com/39436379/42804972-1cd179ba-89dd-11e8-8d6f-4642f463d3ed.png)

Since SPECTRE and PHANTOM are also proof-of-work-based, they too are susceptible to reorgs.   
Each node’s canonical set of transactions (TxO set) is derived from the order of blocks that the protocol produces:   
a node locally computes the order of blocks, then chronologically accepts consistent transactions.   
In a DAG reorg, a successful attacker may change the initially observed order of a node’s local DAG.   
If transaction tx was previously included in the node’s TxO set, an attacker could create blocks that manipulate the order such that tx is no longer part of this set.   
This requires publishing a block with a transaction that conflicts with tx, and having that block precede the block containing tx.

由于SPECTRE和PHANTOM也是基于工作量证明的协议，因此它们也容易受到重组。  
每个节点的规范交易集（TxO集）是由协议产生的块的顺序产生的：  
节点在本地计算块的顺序，然后按时间顺序接受一致的交易。  
在DAG重组中，成功的攻击者可能会更改节点的本地DAG的初始观察顺序。  
如果交易tx之前包含在节点的TxO集中，则攻击者可以创建操纵该顺序的块，使得tx不再是该集合的一部分。  
这需要创建一个块，块中一个交易与tx冲突，并且该块在包含tx的块之前。

![1_h6xa16iehnj27h11dwe8ow](https://user-images.githubusercontent.com/39436379/42805005-37eb113e-89dd-11e8-9647-1e04eff4d4be.png)

![1_9znczkp-tikqyxfocwekkq](https://user-images.githubusercontent.com/39436379/42805029-4712f5fa-89dd-11e8-85f5-3ee202f34eba.png)

>The above figure shows two blockDAGs that a node hypothetically observes at two different times and the DAGs’ corresponding possible orderings (produced via the PHANTOM protocol).   
>With the addition of blocks N and O in the second DAG, PHANTOM will produce a different clustering of “honest” vs. “dishonest” blocks, and thus the ordering is changed drastically. 

>上图显示了假设在两个不同时间，节点观察到的两个blockDAG以及DAG对应的可能排序（通过PHANTOM协议生成）。  
>通过在第二个DAG中添加块N和O，PHANTOM将产生不同的“诚实”与“不诚实”块的集群，因此排序会发生巨大变化。   

>If there are conflicting transactions in, say, blocks I and O, then the previously accepted transaction in block I will no longer be in the node’s TxO set.   
>Note that PHANTOM produces a full linear ordering of blocks, which is simplest to illustrate, whereas SPECTRE produces a pairwise ordering. 

>如果在块I和O中存在冲突的交易，则块I中先前接受的交易将不再出现在节点的TxO集中。  
>请注意，PHANTOM可以生成块的完整线性排序，这是最简单的说明，而SPECTRE只能生成成对排序。

## Confirmation times in Bitcoin and blockDAGs

## 比特币和blockDAG中的确认时间

In Bitcoin, the confirmation time of a transaction is the time it takes for the transaction to become effectively irreversible, which is equivalent to waiting until the block containing the transaction is protected against orphanization from reorgs.

在比特币中，交易的确认时间是交易变得有效且不可逆转所需的时间，这相当于直到包含交易的块被保护免于重排造成的孤块化所需要的等待时间。
   
Nodes set their own threshold for how few confirmations they are willing to accept before recognizing a payment as irreversible.

在支付被确认为不可逆转的之前，节点设置了他们愿意接受的确认数量。
   
To that end, each node must estimate the likelihood of a chain reorg;   
this is done by locally setting two parameters:

为此，每个节点必须估计链重组的可能性; 
这是通过本地设置两个参数来完成的：

$\alpha$ — an assumption on the percentage of the total hashrate an attacker might have
$\varepsilon$ — the risk the node is willing to absorb

$\alpha$ - 对攻击者可能拥有的总哈希值的百分比的假设
$\varepsilon$ - 节点愿意承担的风险

A typical Bitcoin client will consider a transaction confirmed only once it is six blocks deep   
(that’s an average confirmation time of 60 minutes!).   
This default number is based on an attacker size of 10% and a risk of <0.1%.²

一个典型的比特币客户端认为只有在六个块深度的交易才能被确认  
（这是60分钟是平均确认时间！）。  
此默认数字基于攻击者比例大小为10％和风险<0.1％。²

Similarly, a node in a blockDAG system must locally set some parameters in order to estimate the robustness of its current ordering of the DAG.   
In addition to the parameters $\varepsilon$ and $\alpha$ needed for confirming transactions in Bitcoin, the node specifies an assumption on the network delay D.   
Assumptions on the network propagation delay are essential when characterizing consensus protocols.   

类似地，blockDAG系统中的节点必须在本地设置一些参数，以便估计其当前DAG排序的稳健性。  
除了确认比特币交易所需的参数$\varepsilon$和$\alpha$之外，该节点还指定了网络延迟的假设(值)D。  
在表示共识协议的特征时，对网络传播延迟的假设是必不可少的。

The nature of this assumption may differ across protocols, e.g., where in the protocol it is made, whether it is part of consensus, and what harm is done when it is violated and for how long.   
The Bitcoin protocol, for instance, operates under the (rather conservative) ten-minute upper bound on the network delay.³

这种假设的性质可能因协议而异，例如，在协议中的位置，是否是共识的一部分，和在违反协议时产生多少伤害以及持续多长时间。  
例如，比特币协议在网络延迟的（相当保守的）十分钟上限下运行。³

### Size of attacker in blockDAGs

### blockDAG中攻击者的比例

An interesting implication of having no orphans in a blockDAG system is that reorgs affect only conflicting transactions, and so transactions with no visible conflicts remain unaffected.   

在blockDAG系统中没有孤块的一个有趣的含义是，重组只影响冲突的交易，因此没有可见冲突的交易不会受到影响。

In particular, a double-spending attacker must engage directly with its victims, broadcasting a conflicting transaction for each transaction it aims at reversing.   

特别是，双花的攻击者必须直接与受害者接触，为旨在扭转链的每笔交易广播与其冲突的交易。

In contrast, a powerful attacker in Bitcoin can harm a huge number of payments by publishing a sufficiently long chain of empty blocks.

相比之下，比特币中强大的攻击者可以通过发布足够长的空块链来是大量支付受损。

Consequently, sometimes $\alpha$  can be set to be quite small in a blockDAG system, which significantly reduces confirmation times.   

因此，有时在blockDAG系统中$\alpha$可以设置得非常小，这大大减少了确认时间。

For instance, in the case of physical point-of-sale payments, it is unlikely that a large mining entity will bother to visit the point of sale and reverse the payment.   

例如，在实际销售点支付的情况下，大型挖矿实体不太可能费心去访问销售点并撤销支付。

Point-of-sale nodes may therefore justifiably set their $\alpha$ to be small — maybe even 0 — allowing them to confirm transactions much faster.

因此，销售点节点可以合理地将其$\alpha$设置是小的 - 甚至可以为0  - 允许它们更快地确认交易。

## Confirmation times, the SPECTRE case

## 确认时间，SPECTRE案例

In SPECTRE, the assignment of D is local to the node running the protocol; there does not need to be agreement.

在SPECTRE中，对于运行该协议的节点D的分配是局部的；不需要是一致的。  
   
This also does not need to be a correct estimate, as knowledge of network propagation delays is tricky and dynamic. 

这也不需要有正确的估计，因为对网络传播延迟的了解是棘手的，而且网络传播延迟是动态的。
  
Instead, each node sets an upper bound on D based on recent history. This is analogous to the upper-bound network delay estimate upon which Bitcoin’s ten-minute block time is based.
  
相反，每个节点根据最近的历史记录在D上设置上限。这类似于比特币的十分钟块时间所基于的上限网络延迟估计。

Unlike in Bitcoin, each node in SPECTRE solely bears the consequences of its choice of D.   

与比特币不同，SPECTRE中的每个节点仅承担其选择D的后果。

A paranoid node that significantly overestimates D will hurt itself by waiting too long to recognize that a transaction is irreversible, and a negligent node that significantly underestimates D will hurt itself by prematurely accepting transactions. 

一个明显过高估计D的偏执节点会因为等待太长时间，而无法识别交易是不可逆转的而损害自身，而一个明显低估D的疏忽节点会因过早接受交易而损害自身。

However, nodes may change their choice of D as they wish, e.g., if their former choice is overly inaccurate or if network conditions change — and they can do so without coordinating with the rest of the network.

然而，节点可以根据需要改变他们对D的选择，例如，如果他们以前的选择过于不准确或者网络条件改变 - 并且他们可以在不与网络的其余部分协调的情况下这样做。

Thus, the core protocol of SPECTRE (i.e., the pairwise ordering procedure) is agnostic to the network propagation delay.  
Confirmation times in SPECTRE do largely depend on the propagation delay, but this is estimated locally by each node.

因此，SPECTRE的核心协议（即成对排序过程）对网络传播延迟是不可知的。  
SPECTRE中的确认时间很大程度上取决于传播延迟，但这是由每个节点在本地估计的。

Asymptotically, SPECTRE’s confirmation times are in

渐渐地，SPECTRE的确认时间是

![1_ea_z18eack9wt9atnz8f5a](https://user-images.githubusercontent.com/39436379/42856903-dff75ce2-8a79-11e8-9b55-b26a5ce078d6.png)

where $\lambda$ is the block creation rate.⁴   
Under a range of normal network conditions, this yields confirmation times on the order of seconds.   
When the network is healthy, nodes will tighten their upper bound on D and minimize this time.   
The effects of D and $\lambda$ on confirmation times are shown in the simulation results below.

其中$\lambda$是出块率。⁴  
在一组正常的网络状况下，这就产生了以秒为单位的确认时间。  
当网络健康时，节点将缩小D的上限并使确认时间最小化。  
D和$\lambda$对确认时间的影响在下面的模拟结果中显示。

![1_l1q7ctpfucygxzt-bg7t0q](https://user-images.githubusercontent.com/39436379/42856957-20e8d79e-8a7a-11e8-813d-2f44ad583f0c.png)

>(left) Effect of delay D on confirmation times with $\lambda = 10$ blocks per second and $\alpha = 0.25$. 
>
>延迟D对确认时间的影响,每秒$\lambda =10$块和$\alpha = 0.25$。
>(right) Effect of block creation rate $\lambda$ on waiting time with D = 5 seconds and $\alpha = 0.25$. Images from Appendix B of the SPECTRE paper.   
>(Note: as stated earlier, in some cases nodes may safely set $\alpha$ as low as 0, speeding up confirmation times even further.)

>(右图)出块率$\lambda$对等待时间的影响，D = 5秒且$\alpha = 0.25$。图片来自SPECTRE论文附录B。  
>(注意：如前所述，在某些情况下，节点可以安全地将$\alpha$设置为低至0，从而进一步加快确认时间。)

## Weak Liveness in SPECTRE

## SPECTRE中的弱活性

SPECTRE is able to achieve fast confirmations by relaxing the Liveness property which, together with Safety, is traditionally required from consensus protocols.   
In this context, Safety means that a decision is not reversed w.h.p., and Liveness means that a decision is guaranteed to be made (preferably sooner rather than later).   

通过调节活性，SPECTRE能够实现快速确认，从传统上来说，共识协议需要该属性与安全性。  
在这种情况下，安全意味着决定不会逆转w.h.p.，而活性意味着保证做出决定（最好是早点而不是晚点）。

In SPECTRE, if two conflicting transactions are published at the same time or one soon after another, then neither transaction is guaranteed to be effectively irreversible and thus decided upon within a finite time.   
On the other hand, a published transaction with no near-published conflicts is guaranteed to become irreversible.   
The authors of SPECTRE call this property Weak Liveness.

在SPECTRE中，如果两个冲突的交易同时发布或一个接一个地发布，那么这两个交易都不能保证有效不可逆转，因此在有限的时间内决定。  
另一方面，没有发布冲突的已发布交易将保证是不可逆转的。  
SPECTRE的作者将此属性称为弱活性。

It is crucial to observe that this weakening does not harm the usability of the system for honest users.   
Honest users would never attempt to double spend a transaction right after they have published it, and so for such users the weakening of Liveness is non-consequential.   
Dishonest users, in contrast, might publish conflicting transactions;   
for these users Weak Liveness implies that both transactions might remain pending for an indefinite period of time.⁵   
Importantly, recipients of these published conflicting transactions will immediately notice the double spend attempt and refrain from accepting either transaction as valid.

至关重要的是要观察到这种削弱不会损害系统对诚实用户的可用性。  
诚实的用户永远不会尝试在发布交易后立即加倍交易，因此对于这些用户来说，活性的削弱是无关紧要的。  
相反，不诚实的用户可能会发布冲突的交易;  
对于这些用户而言，弱活动意味着两个交易可能会无限期地保持未决状态。⁵  
重要的是，这些发布的冲突交易的接收者将立即注意到双花尝试，并且不会接受任何一个交易并视为有效。

Note that this argument applies only to a payments use case, for which SPECTRE is designed, in which the owner of the funds is the sole entity that can produce and sign such conflicting transactions.   
In contrast, in a smart contracts application, any user could potentially feed the contract with conflicting inputs; SPECTRE is less suited for this.

请注意，此参数仅适用于SPECTRE的付款方案，其中资金所有者是可以生成和签署此类冲突交易的唯一实体。  
相反，在智能合约应用程序中，任何用户都可能使用冲突的输入来提供合约; SPECTRE不太适合这种情况。

In PHANTOM, some of these issues are dealt with in a new way, as we will discuss in future blog posts.

在PHANTOM中，其中一些问题以新的方式处理，我们将在未来的博客文章中讨论。

## Endnotes:
¹ This is a term borrowed from Elaine Shi and Rafael Pass.

¹ 这是从Elaine Shi和Rafael Pass处引用的术语。

² Source: https://bitcoil.co.il/Doublespend.pdf

³ Technically, the estimate of ten minutes is more subtle: it is an assumption that the probability of a Poisson process with mean interval of ten minutes producing two events within an interval shorter than D is negligible.

³ 从技术上讲，十分钟的估计更为微妙：假设平均间隔为十分钟的泊松过程在短于D的区间内产生两个事件的概率可以忽略不计。

⁴ Derivation of the asymptotic bound (LaTeX’d for math formatting):

⁴ 渐近边界的推导（用于数学格式的LaTeX格式）：

![1_yfwl27gldbihmdh8gmo9fw](https://user-images.githubusercontent.com/39436379/42867070-0810c00c-8aa1-11e8-9648-e4ea515edad1.png)

双花的可能性可以大致表示为$\left (  \frac{\alpha }{1-\alpha }\right )^{n-m-K}$,n是诚实块的数量，m是被隐藏/已显示的攻击者块的数量，K是在攻击阶段在边缘创建的块的数量，e.g.，没有通过我们试图防御的块的路径连接的块。渐渐地，m表现为$n*\frac{\alpha }{1-\alpha }$,
K表现为泊松过程，其平均值与D成比例$D*\lambda$(写作：$C*D*\lambda$)。风险小于$\varepsilon$所需的诚实区块的数量是$O(\frac{ln(\varepsilon) }{ln(\alpha /(1-\alpha ))}*\frac{1-\alpha }{1-2\alpha }+C*D*\lambda)$.为了得到期望的确认时间，我们把这个块的数量与诚实矿工出块的期望时间，$\frac{1}{(1-\alpha )*\lambda }$，相乘。我们得到$O(\frac{ln(\varepsilon) }{ln(\alpha /(1-\alpha ))}*\frac{1}{(1-2\alpha)*\lambda  }+C*\frac{D}{1-\alpha })$,简化为$O(\frac{ln(\varepsilon) }{(1-2\alpha )*\lambda }+\frac{D}{1-\alpha })$。

⁵ Even this only happens under peculiar attacks coordinated by malicious miners.

⁵ 即使这只发生在恶意的合作矿工的特殊攻击下。
