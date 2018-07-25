>Source:https://blog.daglabs.com/transaction-selection-games-in-blockdags-602177f0f726

>TranStudy：https://github.com/DAGfans/TranStudy/edit/master/Blogs/Transaction%20Selection%20Games%20in%20BlockDAGs.md


# Transaction Selection Games in BlockDAGs

# BlockDAG中的交易选择博弈

*Consider a blockDAG network which mines 1 block per second, and an individual Miner A within it.   
How will Miner A know which transactions to embed in his next block?   
Obviously, he can fill his block with the highest-paying transactions, but then he faces the risk that these transactions simply duplicate the transactions that Miner B includes in his block, created at the same time in a different faction of the network.   
Miner A — and by symmetry Miner B — may thus consider being less greedy, selecting lower-paying fees, and avoiding the duplications, or “collisions”.   
In fact, this game is similar to the famous game of Chicken (aka Hawk–Dove Game), in which avoiding collisions is in the best interest of both parties.   
Interestingly, if we implement the Nash equilibrium strategy in the default mining client, an individual miner will not benefit from deviating, applying a more “Hawkish” strategy, and attempting to collect the highest-paying transactions.   
In this post we discuss how this affects a blockDAG’s throughput and quality of service for transaction confirmation.*

*思考下有一个blockDAG网络，它每秒挖出1个块，并且在其中有一个矿工A。
矿工A将如何知道在下一个区块中嵌入哪些交易？
显然，他可以用最高手续费的交易填满他的块，但是他面临着这些交易只是复制矿工B在其区块中包含的交易的风险，交易同时在网络的不同阵营中创建。
矿工A——以及与其行为一致的矿工B——因此可以考虑不那么贪婪，选择手续费较低的交易，避免重复或“冲突”。
事实上，这个博弈类似于着名的胆小鬼博弈（又名Hawk-Dove Game），其中避免碰撞符合双方的最佳利益。
有趣的是，如果我们在默认的挖矿客户端中实施纳什均衡策略，矿工不会因违背均衡而受益，采用更“强硬”的策略，并试图收集手续费最高的交易。
在这篇文章中，我们讨论了这会如何影响blockDAG的吞吐量和交易确认的服务质量。*

![1_n3fhm8ymvlv12xhvypdt0a](https://user-images.githubusercontent.com/39436379/42679444-8820eea2-86b4-11e8-92c5-f9dc41ff43b7.gif)

## Background

## 背景

Since blockDAGs can operate at very high block creation rates, multiple blocks may be created in parallel in the network.   
Recall that all blocks in a blockDAG are included in the ledger, without being orphaned, and thus there is potential for a large throughput increase over Bitcoin.   
However, miners mining blocks in parallel to each other cannot directly coordinate with each other, and if they choose the same subset of transactions to include in their blocks, there would be many transaction collisions and throughput would be wasted (though confirmation times would still be significantly faster).

由于blockDAG有非常高的出块率，因此可以在网络中并行创建多个块。
回想一下，blockDAG中的所有块都包含在账本中，而不会成为孤块，因此有可能超过比特币而大量增加吞吐量。
但是，矿工彼此并行挖矿，不能直接相互协调，如果他们选择相同的交易子集包含在他们的块中，那么会有很多交易碰撞，浪费吞吐量（虽然确认时间仍然会明显加快）。

## Incentives in transaction selection

## 交易选择中的激励

The Inclusive Blockchain Protocols paper, presented at Financial Crypto 2015, highlights an important insight: miners are incentivized to avoid transaction collisions.   
A transaction fee is produced only once, and can be given only once (or split), even if the transaction is duplicated across multiple blocks.   
Assuming that the winner will be dictated by the ordering protocol applied on the DAG, each block has some probability of collecting the fee of a transaction embedded in it and some probability of losing it due to collisions.   
Miners are therefore incentivized to minimize collisions and hence increase the DAG’s utilization and throughput.

2015年Financial Crypto上发表的“Inclusive区块链协议”文章强调了一个重要的见解：激励矿工避免交易碰撞。
交易费只产生一次，并且只能给出一次（或拆分），即使交易是跨多个块重复的。
假设获胜者将由DAG上应用的排序协议决定，每个块都有可能收取嵌入其中的交易的手续费，并且有一定概率因碰撞而失去手续费。
因此，鼓励矿工最大限度地减少碰撞，从而提高DAG的利用率和吞吐量。

The Inclusive authors go further to model this dynamic as a game of transaction selection.   
In the game, each miner needs to strategically decide which transactions to include in his next block.   
Interestingly, choosing the highest-paying transaction does not usually maximize the miner’s profit. The matrix below describes a simple game, as an example. 
It specifies the expected payoffs in the game where two miners need to choose one transaction for their next block, from a mempool of two transactions with different fees.   
Notice the resemblance to the game of Chicken, where if one player commits to the more rewarding choice, the other is incentivized to sway and choose the less rewarding one.¹

作者进一步将这种动态模型转化为交易选择博弈。
在博弈中，每个矿工都需要有策略地决定在下一个区块中包含哪些交易。
有趣的是，选择手续费最高的交易通常不会使矿工的利润最大化。
下面的矩阵描述了一个简单的博弈，作为一个例子。
它规定了游戏中的预期收益，其中两个矿工需要为下一个区块选择一个交易，来自内存池中两个不同手续费的交易。
注意与胆小鬼博弈的相似之处，如果一个玩家作出了更有价值的选择，另一个玩家会被激励退缩从而选择奖励较少的选项。¹

![1_mrcomzov5mwcxwhukff9la](https://user-images.githubusercontent.com/39436379/42679492-b1f333e8-86b4-11e8-9886-8b4fdbec5ffa.png)

>Miner1 and miner2 have tx1 and tx2 in their mempools. For the next transaction they decide to include, if they collide on the same transaction then they each only have a (roughly) 50% chance of getting the full transaction fee. They are thus incentivized to choose the transaction that the other does not choose — if the difference between the fees is not too large. However, they cannot coordinate their strategies.

>矿工1和矿工2在他们的内存池中分别有交易1和交易2。
>对于他们决定包含的下一笔交易，如果他们发生碰撞，那么他们每人只有（大约）50％的机会获得全部交易费。
>因此，他们被激励去选择别人没有选择的交易——如果它们之间的费用差异不是太大。
>但是，他们无法协调他们的策略。

A player adopts a mixed strategy if he defines a probability distribution that assigns a likelihood to each of his possible actions in a game; 
thus, in this game, a miner’s mixed strategy is a probability distribution that defines how miners choose transactions to include in their next block.   
This game is described in detail in the Inclusive paper, where several solution concepts are analyzed, including the well known Nash equilibrium concept (see section 4.3).   
If Nash is played by all miners, then by definition no individual miner benefits from deviating.   
Practically, we can embed this strategy in our default mining client, which makes it easier to reason about how the equilibrium was reached.

如果玩家定义了一个为博弈中的每个可能动作都分配了概率的概率分布，则玩家采用混合策略；因此，在这场博弈中，矿工的混合策略是一种概率分布，它定义了矿工如何选择要包含在下一个区块中的交易。
这个游戏在论文中有详细描述，其中分析了几个解决方案，包括众所周知的纳什均衡概念（参见4.3节）。  
如果纳什均衡是由所有矿工参与的，那么根据定义，任何一个矿工都不会因违背均衡而受益。  
实际上，我们可以将此策略嵌入到我们的默认挖矿客户端中，这样可以更容易地推断出均衡是如何实现的。

## Tradeoffs in transaction selection
## 交易选择的权衡

The throughput of a blockDAG is highly dependent on the strategy profile in this transaction selection game.   
For example, if each miner always chooses the k highest-fee transactions with probability 1 (where k = number of transactions per block), then parallel blocks are likely to contain many collisions, as miners of these blocks will choose the same subset of transactions from the mempool.   
In this case, the blockDAG would not enjoy any throughput improvement over a blockchain (though, again, confirmation times would still be much faster).

blockDAG的吞吐量高度依赖于此交易选择博弈的策略。
例如，如果每个矿工总是选择概率为1最高手续费为k的交易（k=每个块的交易数），那么并行块可能包含许多碰撞，因为这些区块的矿工将从内存池中选择相同的交易子集。  
在这种情况下，blockDAG相比区块链不会有任何吞吐量改进（但同样，确认时间仍然会快得多）。

On the other hand, if each miner chooses transactions uniformly from the mempool (above some small fee threshold), then there will be almost no collisions and throughput will be optimized.   
However, transactions will be served uniformly slowly, and those with higher fees will not be prioritized.

另一方面，如果每个矿工从内存池中统一选择交易（超过一些小额手续费门槛），那么几乎没有碰撞，吞吐量将得到优化。但是，交易将统一缓慢地进行，而那些手续费较高的交易将不会被优先考虑。

More generally, the transaction selection strategy faces a tradeoff between high quality of service for confirmation times and high throughput.   
Furthermore, the chosen strategy depends on how cooperative miners are, and how much they expect to benefit from malicious deviation.   
For example, to achieve optimal throughput via the uniform selection strategy, miners have to almost entirely disregard fee differences, giving up on profit maximization.

一般地说，交易选择策略面临在确认时间的高质量服务和高吞吐量之间权衡。此外，所选策略取决于合作矿工的合作程度，以及他们希望从恶意违背中获益的程度。例如，为了通过统一选择策略实现最佳吞吐量，矿工几乎只能完全忽视手续费差异，放弃利润最大化。

![1_jsxw_h6bh03jk-8joy5t1a](https://user-images.githubusercontent.com/39436379/42682463-407a1d18-86bd-11e8-86d6-ec03d87f0cab.png)

The graph above depicts the confirmation times of two different transaction selection strategies, which differ in the weight they give to the value of the fee.   
The orange curve represents a strategy that prioritizes high-fee transactions by assigning them higher inclusion probability in the next block.   
Those that are willing to pay the high fees will get confirmed fast, but there will be more collisions and the fee threshold to be selected from the mempool will be higher.   
The blue curve, in contrast, represents a more egalitarian strategy:   
it randomizes more uniformly over the mempool, assigns similar inclusion probability to transactions with medium-to-high fees, and therefore serves them roughly after the same waiting time.   
This more egalitarian strategy enjoys fewer collisions and higher throughput.   
However, urgent transactions might not be able to “buy” faster confirmation times with higher fees.

上图描绘了两种不同交易选择策略的确认时间，它们对手续费的权重不同。  
橙色曲线表示的策略是通过在下一个区块中为其分配更高的包含概率来优先处理高手续费交易。那些愿意支付高额手续费的交易将得到快速确认，但会有更多的碰撞，从内存池中选择的费用门槛会更高。  
相比之下，蓝色曲线代表了一种更平等的策略：它在内存池上更均匀地随机化，为具有中到高费用的交易分配相似的包含概率，因此大致在相同的等待时间之后为它们服务。  
这种更平等的策略享有更少的碰撞和更高的吞吐量。  
但是，紧急交易可能无法以更高的手续费“购买”更快的确认时间。

Fortunately, the Inclusive authors show that the trade-off is not severe — their Nash equilibrium solution achieves both high throughput and high quality of service levels.   
This solution assigns high probability to high-fee transactions, but these transactions are not selected with complete certainty, as some weight is given to lower-fee transactions.   
The authors ran simulations comparing the throughput of a blockDAG protocol with the Nash strategy profile to that of a non-inclusive longest-chain protocol.   
Though it does not reach optimal utilization, the blockDAG achieves high throughput (proportional to optimal throughput, in fact), while still maintaining the ability to prioritize high-paying users.

幸运的是，论文的作者表明权衡并不严重--他们的纳什均衡解决方案实现了高吞吐量和高质量的服务水平。  
该解决方案为高手续费的交易分配了高概率，但这些交易未必一定会被选中，因为对较低手续费的交易也给予了一定的权重。  
作者进行了模拟，将blockDAG协议，纳什均衡策略与非包含性最长链协议的吞吐量进行了比较。  
虽然它没有达到最佳利用率，但blockDAG实现了高吞吐量（实际上与最佳吞吐量成比例），同时仍然保持了对支付高手续费用户进行优先级排序的能力。

![1_cadzsrqt9kxvzxwuatfiga](https://user-images.githubusercontent.com/39436379/42683193-31ec0f70-86bf-11e8-9c3e-56d9b10b1255.png)

>Simulation results from the Inclusive paper (section 5.1): “We simulated a network with 100 identical players. The distance between each pair of players was a constant d = 1 second. We examined different block creation rates λ varying from 0 to 10 blocks per second. Block sizes were set to b = 50 transactions per block. The transaction arrival rate was 65 transactions per second, and their fees drawn uniformly from [0,1].”
>Inclusive论文（第5.1节）的模拟结果：“我们模拟了一个拥有100个相同用户的网络。每对用户之间的时间间隔是（常数）d=1秒。我们检查了不同的块出块率λ，从每秒0个块到每秒10个块。块大小设置为每块b=50个交易。交易确认率为每秒65笔交易，其手续费统一在[0,1]区间。

## Conclusion

## 结论

Protocol scalability is often measured in terms of throughput (transactions per second) and latency (confirmation times). While blockDAG protocols can achieve significantly faster confirmation times than blockchains, they typically face a tradeoff between quality of service for confirmation times and throughput, as demonstrated above.   
The tradeoff a blockDAG protocol takes depends on the behavior of miners when selecting transactions from their mempool to include in their next block. 
This behavior induces a non-trivial transaction selection game, in which miners are often incentivized to avoid duplications (or “collisions”), as these lower the chances of winning the fees. 
Still, transactions with high fees are selected by many miners, regardless of the collision risk, which allows for decent quality of service. 
Fortunately, these incentives imply that miners would naturally contribute and work towards higher utilization and throughput of the system, even when considering only their own self-interests.

协议的可伸缩性通常根据吞吐量（每秒处理交易数）和延迟（确认时间）来衡量。  
综上所述，虽然blockDAG协议相比区块链可以得到明显更快的确认时间，但它们通常在确认时间和吞吐量的服务质量之间进行权衡。blockDAG协议的权衡取决于矿工从他们的内存池中选择交易以包含在他们的下一个区块时的行为。
这种行为导致了一个与以往不同的的交易选择博弈，矿工经常被激励以避免重复（或“碰撞”），因为这会降低赢得手续费的机会。
尽管如此，许多矿工都选择了高手续费的交易，无论碰撞风险如何，目的是提供良好的服务质量。  
幸运的是，这些激励措施意味着矿工自然会为系统的更高利用率和吞吐量做出贡献并努力，即使只考虑自己的自身利益。 
