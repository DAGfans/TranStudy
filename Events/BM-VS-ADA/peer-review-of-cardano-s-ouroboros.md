
原文： https://steemit.com/cardamon/@dan/peer-review-of-cardano-s-ouroboros  
作者： [Daniel Larimer](https://steemit.com/@dan)

声明：DPOS系列的项目和ADA都是我认为非常优秀的项目，学习这篇文章仅仅希望从当事方的角度去深入了解两者的技术上的区别，以后ADA官方若有正式回应我也会第一时间跟进

I recently had an opportunity to review Cardano’s Ouroboros consensus algorithm as presented in a youtube presentation. The original paper can be found here. The marketing behind Cardano and Ouroboros is that it is the first “peer reviewed”, “provably secure” proof of stake consensus algorithm. Upon reading their paper it becomes clear to those familiar with BitShares 1.0, Graphene, Steem, and EOS that Ouroboros is a copy of Delegated Proof of Stake (DPoS) with a few counter-productive modifications. In fact their paper refers to the term “πDPoS” 17 times without mentioning or recognizing any of my prior work.  
我最近有机会评审了一下youtube演示文稿中介绍的Cardano的Ouroboros共识算法。 原文可以在这里找到。 Cardano和Ouroboros背后的营销方案说它是第一个“同行评审”，“可证明安全”的股权证明共识算法。 在阅读他们的论文时，对于那些熟悉BitShares 1.0，Graphene，Steem和EOS的人来说，Ouroboros是一个委托股权证明（DPoS）的复制品，而且有一些适得其反的修改。 事实上，他们的论文中有17次提到了“πDPoS”，却没有提到或承认我以前的任何工作。


Before getting into the more technical review of Ouroboros, let’s look at the performance of the final product.  
在进行Ouroboros更多的技术评审之前，让我们先看看最终产品的性能。


Block Interval  
区块周期  
Block interval determines the latency until a transaction is included in the first block. This is the lower-bound on the responsiveness of decentralized applications built on the protocol. Applications like Steem and BitShares are not really viable unless there is low latency and high certainty of finality.  
区块周期决定了交易第一次打包进区块的延迟时间。 这是分部式应用程序在网络协议上的响应时间的最低值。 像Steem和BitShares这样的应用程序如果没有低延迟和高确定性的终结性，是不可行的。

EOS: 0.5 秒
Steem/BitShares: 3 秒
Ouroboros: 20 秒

Irreversibility  
不可逆性  
This is how long someone must wait to be certain that a transaction will not be undone by a new/longer fork being released that excludes the transaction. Irreversibility is very important for any multi-step transactions. You don’t want to ship goods until payment is confirmed. You cannot make one trade until the prior trade is locked in. A decentralized exchange is not viable on a platform that has significant latency until irreversibility.  
不可逆性是指用户需要等待多长时间才能保证交易不会被更新的或者更长的不包含这笔交易的分叉取消。不可逆性对于任何多步骤交易都非常重要。 因为你肯定不希望在确认收款前发货。 你不能在之前的交易锁定时发起新的交易。分布式交易所在不可逆前会有很大的延迟的平台上是不可行的。

EOS: <= 2 秒
Steem/BitShares: <= 45 秒
Ouroboros: > 5 小时

Ouroboros is Unfit for Decentralized Applications  
Ouroboros 是不适合做分布式应用的  
If we assume Ouroboros is actually “more provably secure” by some definition of secure, it is of little practical value because as specified the security completely compromises the practically. It would be like claiming a bullet proof vest is “provably safe” but it weighs 400 pounds. At some point other factors of system design take priority.  
如果我们假设Ouroboros实际上是通过一些安全的定义来“证明更安全”，那么它就没有什么实际价值，因为是为了安全性而牺牲了实用性。 这就像要求一件防弹背心“可以证明很安全”，但重达400磅。 在某些时候，系统设计中的其他因素是应该优先考虑的。  

Unfortunately we cannot simply assume it has been proven secure. I will demonstrate that despite claims to the contrary, Ouroboros is far less secure due to faulty assumptions in its design. In other words, Ouroboros is a 400 pound bullet proof vest that doesn’t actually stop the real bullets.  
但是不幸的是，我们也不能简单地认为它已被证明是安全的。 我将证明，尽管有相反的要求，但由于其设计中的错误假设，Ouroboros远没有那么安全。 换句话说，Ouroboros虽然是400磅的防弹背心，但实际上并不能阻止真正的子弹。

Components of Delegated Proof of Stake (aka Ouroboros)  
委托股权证明的组成部分(即 Ouroboros)  
The Delegated Proof of Stake algorithm is divided into two parts:  
委托权益证明算法分成两部分:  
Selecting a set of block producers  
选择一组区块生产者  
Schedting Block Producers  
选择区块生产者  
Under Ouroboros anyone who wants to be a producer can be selected for a slot proportional to the amount of stake they own or is delegated to them. This is similar to how Steem selects 1 producer in every set (aka every minute). Whereas Steem only schedules 1 out of every 21 time slots in this manner, Ouroboros schedules all time slots with this mathematical distribution. In the case of Ouroboros only those with at least 1% of stake delegated to them are eligible to be in the set of producers, but Steem places no lower limit.  
在Ouroboros中，任何人想要成为生产者都可以选择一个与他们拥有或被委托股份的数量成正比的时段。 这与Steem如何在每一组中选择一个生产者（也就是每分钟）类似。 Steem以这种方式在每21个时段中只会安排一个时段（上的生产者），而Ouroboros则用在数学上分布来安排所有的时段（上的生产者）。 在Ouroboros的情况下，只有那些授权给他们至少1％股份的人才有资格成为出块者，但是Steem没有最低限度。



Scheduling Block Producers into Slots  
安排区块生产者到时段内  
The primary difference between Steem and Ouroboros’ system is that Steem uses deterministic scheduling with pseudorandom shuffling and Ouroboros uses sampling from a source of provable randomness created by a committee of randomly selected stakeholders. Oroboros places a very strong focus on the need for a trustless source of randomness in order to ensure the producer schedule isn’t manipulated by block producers manipulating the block contents so as to control the schedule.  
Steem和Ouroboros系统之间的主要区别在于Steem使用确定性调度和伪随机乱序机制，而由随机选择的股份持有者组成的委员会创建可证明的随机性来源后，Oroboros会从中抽样。 Oroboros非常依赖不可信源的随机性，以保证出块计划不会被区块生产者通过操纵区块内容而被控制。


Security Concerns addressed by Randomness  
随机性解决安全问题  
The heart of blockchain security is knowing that during every confirmation window that a diverse set of unlikely-to-collude entities produce blocks. Time to irreversibility depends upon how long it takes to get input from a 2/3+ majority of potential producers.  
区块链安全的核心是知道在每一个确认窗口中，一组不太可能串通的实体会产生区块。 不可逆的时间取决于需要多长时间才能从2/3以上的潜在生产者中获得输入。

If the order of producer scheduling can be controlled by the producers there exists a circular dependency. This is the origin of the name Ouroboros (a snake eating its own tail). The focus of the Ouroboros design is to ensure that 2/3+ honest producers can be scheduled without interference. This is why they lock in the set of producers for days and schedule them with a set of provable randomness that no party can manipulate.  
如果生产者安排的顺序可以由生产者控制，则存在循环依赖。 这是Ouroboros这个名字的起源（一条吃自己的尾巴的蛇）。 Ouroboros设计的重点是确保2/3以上的诚实生产者在不受干扰的情况下被安排。 这就是为什么他们会将一组生产者锁定几天，并通过一套可证明的无人可以操纵的随机性机制来安排。

Steem / BitShares / EOS
Existing DPOS chains select a set of unlikely to collude entities by approval voting and then schedule them in a pseudorandom order. This shuffling is not really needed because once each of them participates a single block a 2/3+ consensus can be determined. This is why EOS will be removing the random shuffle all together.  
现有的DPOS链通过批准投票来选择一组不太可能串通的实体，然后以伪随机顺序安排它们。 这种乱序机制并不是真的必要，因为一旦他们中的每一个参与了一个区块，2/3的共识就已经确定了。 这就是为什么EOS将完全去除随机乱序机制的原因。



With Ouroboros the length of time until 2/3+ of the stake is “randomly selected” is not known. It is entirely possible that in some windows all block producer slots will be randomly assigned to the same producer. While this is statistically unlikely, it is not unreasonable to presume that a long sequence of blocks could be assigned to collusive peers.   
在Ouroboros中，直到2/3以上的股份被“随机选择”的时间的长度是没法确定的。完全有可能在一些窗口中，所有的区块生产者时段将被随机分配给同一个生产者。 虽然这在统计上是不太可能的，但可以假定是将一长串的连续区块分配给串通好的同伴，这并不是不合理的。

You can think of the process of confirmation on Ouroboros to be like an installation progress bar that jumps by random increments. Sometimes it moves forward quickly, other times it taunts you by not making any real progress despite new blocks being produced.  
您可以将Ouroboros上的确认过程想象为以随机的增量跳变的安装进度条。 有时候，它会快速增加，但有时候它也会跟你开玩笑，尽管有新的区块被生产出来，但进度条却纹丝不动。

This gives Ouroboros unpredictable latency like Bitcoin. In the best case 2/3+ of the stake might get scheduled in the a half dozen blocks, but in the average case it will take much longer, particularly if there are many producers with 1% weight or a producer with 50% weight is scheduled many times in a row by random chance.  
这使Ouroboros的延迟跟比特币一样变得无法预知。 在最好的情况下，2/3以上的股份可能被安排在六个区块内，但在平均情况下会需要更长的时间，特别是有很多生产者的权重是1％或单个生产者权重是50%的情况，会被随机多次调度。


Distribution Security Issues
（股权）分布的安全问题
I have previously made the case that BitShares, Steem, and EOS are the most decentralized because it has the most unique confirmations per confirmation window. In a 6 block confirmation window for Bitcoin only 5 unique individuals confirm a block on average (usually at least one mining pool goes twice in a random 6 block window). In Steem 14 people confirm a block each round.  
之前我曾经提到BitShares，Steem和EOS是最去中心的，因为它们在每个确认窗口中拥有最多的唯一（生产者）的确认。 在比特币的6个区块确认窗口中，只有平均5个唯一的生产者确认一个区块（通常至少一个矿池在随机的6个区块窗口中会出现两次）。 在Steem中， 每轮14人确认一个区块。

Because stake and votes are distributed by pareto principle, we know that Ouroboros will assign block producers like mining assigns mining pools. It is exceedingly unlikely that 100 producers will each have 1% of the delegated stake. It is far more likely that there will be fewer than 20 individuals with more than 1% stake required to be approved. Furthermore, there are voting paradoxes in play where voting for someone who doesn’t already have 1% or more of the stake is a wasted vote.  
由于股权和投票是按照pareto原理（80-20原理）分配的，我们知道，Ouroboros会分配区块生产者，就像挖矿会分配矿池。 100个生产者每个都有1％的代表股份是极不可能的。 更有可能的是，只有拥有超过1％股份的不到20人才有资格。此外，投票中存在悖论，投票权不足1％及以上的人的投票是无效的。

In past articles on proof of stake I have also shown that even if Ouroboros removed the 1% requirement to participate, it would be economically unviable to cover the cost of operating a node with income from less than 1% of the block rewards. I have also argued in the past that because stake is distributed by pareto principle, and voter selection of candidates is also selected by pareto principle, the resulting distribution of stake among producers is pareto2. In other words, stake-weighted voting creates a very high centralization that can only be countered with approval voting followed by giving the top N equal weight (like BitShares, Steem, and EOS do).    
在过去的关于股权证明的文章中，我也表明，即使Ouroboros取消了1％的参与要求，不到1%的区块收益在经济上也是不可行的。 我以前也曾经说过，由于股权是按照pareto原理来分配的，候选人的投票也是根据pareto原理来选择，所以生产者之间的股权分布是两级pareto原理。 换句话说，股权加权的投票机制会导致非常高的中心化，只有通过给前N个相同权重（像BitShares, Steem, 和 EOS 那样）的投票权才能反制。

Even if the stakeholders divide their stakes and votes to attempt to even out the producer weights, the system is still ultimately under pareto control 1% of the individuals control 51% of the stake. Corruption takes place at the individual level, not the stake level. Furthermore, it is wrong to assume that large stake holders will behave like a group of smaller stakeholders of similar size. Regardless of their stake, they only have one opinion and one interest and therefore the information (good or bad) they contribute to the system is independent of the size of their stake.  
即使股权持有者拆分他们的股份和选票以试图平衡生产者的权重，根据pareto原则，系统仍然最终会导致1％的人控制51％的股份。 腐败发生在个人层面，而不是股权层面。 此外，假定大股东将表现得像一组规模相似的小股东，这是错误的。 不管他们的股权有多少，他们只有一个目的和利益，因此他们为系统贡献的信息（好的或坏的）与其股份的多少无关。  

Conclusion
结论
Cardano’s Ouroboros algorithm is not mathematically secure due to bad assumptions regarding the relationship between stake and individual-judgment being distributed by the pareto principle. Furthemore, their algorithm is not “new” but a less secure slower variation of the DPOS algorithm I originally introduced in April 2014. The authors of the paper failed to cite relevant prior art or to justify why their deviations from existing art are an improvement.  

股权和个人判断之间的分布是遵循pareto原理的，而Cardano的Ouroboros算法在这方面的假设是错误的，因此其在数学上不是安全的。 此外，他们的算法并不是“新”的，而是我在2014年4月最初引入的DPOS算法上的一个低效和低安全性的变种。文章的作者没有引用相关的现有技术或证明为什么他们差异是对现有技术的一种改进。

A blockchain consensus algorithm claiming to value peer review needs to consider who they consider their peers and all such reviews should be public. In the blockchain space, our peers are other blockchain technology companies. From this we can see that DPOS (and variations thereof) is one of the fastest growing consensus algorithms in terms of the number of unique projects choosing it.  

一个声称重视同行评审的区块链共识算法的团队需要考虑谁才是他们的同行，而且所有这些评审都应该公开。在区块链领域，我们的同行是其他区块链技术公司。 由此我们可以看出，在技术被独立项目使用的数量方面，DPOS（及其变种）是增长最快的共识算法之一。

I am going to go a step further and claim that much of the academic research and proofs performed by Cardano’s team only bolsters the support and justification of many core DPOS concepts, even if their approach is suboptimal compared to designs of EOS, BitShares, and Steem.  
我将更进一步说明Cardano团队进行的许多学术研究和证明仅仅是支持了许多DPOS的核心概念，即使它们的方法与EOS，BitShares和Steem的设计相比并不理想。
