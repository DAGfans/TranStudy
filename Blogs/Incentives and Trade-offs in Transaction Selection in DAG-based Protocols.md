> 文本 https://stanford2017.scalingbitcoin.org/transcript/stanford2017/incentives-tradeoffs-transaction-selection-in-dag-based-protocols
>
> 视频 https://stanford2017.scalingbitcoin.org/presentations
> PPT https://stanford2017.scalingbitcoin.org/stanford2017/Day2/Sompolinsky_SB4.pdf

#Incentives and Trade-offs in Transaction Selection in DAG-based Protocols

#DAG协议中交易选择的激励和权衡

Yonatan Sompolinsky (The Hebrew University)
Yonatan Sompolinsky (希伯来大学)

My name is Yonatan Sompolinsky I'm a PhD student in Hebrew University under the supervision of Aviv Zohar which talked previously and also I'm a co-founder and scientist at DAGlabs which is a commercial effort to
implement DAG based protocols and I feel comfortable here to say that we're not doing an ICO ---- it's a regular
company.

我名叫Yonatan Sompolinsky, 现在在希伯来大学就读博士学位, 之前的演讲嘉宾Aviv Zohar就是我的导师, 此外我还是DAGlabs的联合创始人兼科学家. DAGlabs是一家致力于实现基于DAG协议的商业机构, 在此我表示很放松因为我们不会进行ICO -- 这就是家常规的公司.

A little bit of background about DAG based protocols. The background to this work is a line of works we've done in the Hebrew University with my colleagues, again, Yoad Lewenberg and Aviv Zohar, my supervisor.
This idea of block DAG is a modification of the layer 1 of Bitcoin and so it's a generalization of the chain structure of the blockchain. As such , it's orthogonal to any layer 2 solutions - Lightning Network and all micropayment channels, etc.. So we're scaling up layer 1, you are scaling up layer 2, these are complementary solutions.

稍微介绍下基于DAG协议的背景知识. 此项工作是基于一系列我们在希伯来大学和我的同事们的工作, 即Yoad Lewenberg 和我的导师 Aviv Zohar. 区块DAG(block DAG)的概念是对一层的比特币协议的改进, 所以对链式结构的区块链的泛化(generalization).(译注: 这里的泛化是指区块链是block DAG的一种特例, 所以区块链本质上也是DAG). 正因如此, 它对于任何二层解决方案 , 如闪电网络(Lightning Network)和所有的微支付通道等, 都是互不相关的. 所以说, 我们是在做一层扩容, 你们在做二层扩容, 这些都是可以补充进来的方案.

This talk will be focused on an idea that we develop in the Inclusive blockchain paper in 2015 and we kind of revisited these ideas now because we're heading up to to scale up to implement these block DAG protocols.

本次讨论会聚焦在我们2015研究的Inclusive的区块链论文上, 我们算是在回顾这些想法, 因为我们已经在去实现区块DAG的协议了.

##Blockchain vs BlockDAG

##区块链 vs 区块链DAG

The first difference is that in a chain you would maintain a single chain. But in a DAG you have an entire graph of blocks. In a chain, we ignore anything that is off-chain and any data that was not in the chain. In the DAG, you use all of this information, you harvest all information from all blocks. The third implication that in a chain paradigm you are trying to suppress throughput so that spontaneous forks are rare in the system. In a blockDAG system, forks are a common phenomena of the system and many forks are created in parallel.

第一个区别在于在链式结构中你维护的是单一的链. 但在DAG中你拥有的是整个区块组成的图. 在链式结构中, 我们会丢弃掉不在链下和不在链上的数据. 在DAG结构中, 你可以使用所有的信息, 你可以从所有的区块从提取出所有的信息. 第三个影响是在链式模型中你会尝试抑制吞吐量所以并发的分叉在系统中是很少见的. 但是在区块DAG系统中, 分叉是非常普遍的现象而且会有很多分叉是并行创建的.

As you can imagine, this is essentially a matter of information. In a chain paradigm you ignore everything that's
off chain. In a DAG paradigm you use in this entire information and this information can be used for more
scalability more security more fairness. But the fact that is it can be used to these stuff doesn't mean that every blockDAG uses, so just to drop a name, Ethereum's GHOST is mostly about fairness not scalability.

正如你可以想象的, 这本质上是个信息的问题. 在链式模型中你会丢弃所有的离线信息. 而DAG会利用这个完整的信息来增强可扩容性, 安全性和公平性. 不过事实上, 可以利用这些信息并不代表所有的区块DAG会利用, 随便说个例子, 以太坊的GHOST就主要是用于增强公平性而非可扩容性.

##Scaling up layer 1 using DAGs

使用DAG扩展第1层

The first step would be to speed up block rate. Instead of 1 block every 10 minutes or 1 block every 21 seconds, imagine 50 blocks per second or maybe more. That's step one. So we need spontaneous forks of the chain. The second step is to modifying the mining protocol where instead of extending a chain, you want to reference previous blocks and all the tips that it sees, and integrating the forks into one graph so that you have this massive cool graph which his directed acyclic graph. Seems so simple so far.

第一步将提高出块率。相比每10分钟或每21秒生成一个块，设想每秒生成50个块或者更多。这就是第一步。因此我们需要链的自发的分叉。第二步是在你想要引用之前的块以及它看到的所有提示的位置修改挖掘协议，而不是扩展链，并将分叉集成到一个图中，这样你就可以获得这个非常棒的图，即有向无环图形。到目前为止似乎很简单。

The third step is the most crucial, though. Which is extracting a consistent set of transactions from this graph. The graph might contain conflicts and you want to extract from it and order it and make it work. This is the main challenge of a blockDAG effort. We developed the SPECTRE method to deal with this as discussed 2 years ago in Hong Kong. The consistency rule is the main challenge.

不过，第三步是最重要的。从该图中提取一组一致的交易。图可能包含冲突，你希望从中提取冲突并对其进行排序，使其正常工作。这是blockDAG工作的主要挑战。正如两年前在香港讨论的那样，我们提出了SPECTRE方法来解决这个问题。保持交易一致性规则是主要挑战。

<img width="1172" alt="2018-07-12 4 09 31" src="https://user-images.githubusercontent.com/39436379/42671366-35705a3e-8692-11e8-908f-47404f6be305.png">

I want to clarify that blockDAG is not a solution, it's merely a framework on top of which someone can develop on top of. Consequently, not all blockDAGs are that are created equal I mentioned etheruem ues some notion of a DAG partially. You can design other variants. I like to think of DAG vs chain as chain is slow single one lane road in a small town and a DAG is a highway and the first implicaiton of this is that if you don't know what you're doing, you'll have a mess and you have to think carefully about what you are doing with your DAG. So you want an ordered highway, everything is clear, and high throughput. If you are very lucky, or very thoughtful, you can evolve a protocol with fast confirmations at least for some transactions, which is the topic of this talk.

我想说明一下blockDAG不是解决方案，它只是一个框架，人们可以基于它发展。因此，并非所有的blockDAG都是相同的，我提到过以太坊部分地使用了一些DAG的概念。你可以设计其他变形。我认为DAG VS 区块链,就好像区块链是一个小镇的单一低速的车道，而DAG是一条高速公路，这首先意味着如果你不知道你在做什么, 你会弄得一团糟，然后你必须仔细考虑你要在你的DAG上做些什么。所以你想要一个有序的高速公路，一切都很清晰，吞吐量很高。如果你非常幸运或是非常有想法，至少对于某些交易而言，你可以通过快速确认来改进一个协议，这是本次演讲的主题。

To scale up blockchains using DAGs opens up various challenges. The first challenge is the consistency rule. You also have to address confirmations time, how much storage you want miners or full nodes to store and for how long, so think of 1000 tx/second, at least 86 gigabytes per day， do you want to store this forever or not? These are challenges that need to be addressed. Fee structure, fairness, how to maintain decentralization, etc.

使用DAG扩展区块链会带来各种挑战。第一个挑战是一致性规则。你还必须处理确认时间，你希望矿工或全节点存储多少空间以及存储多长时间，设想下每秒1000笔交易，至少86千兆字节每天，你是否想永远存储它们？这些是需要被解决的挑战，收费结构，公平性，如何维持去中心化等。

How do you utilize the DAG fully so that the entire throughput or at least we're close to the entire throughput that the DAG can supply? I will show you why this is not a trivial challenge. We have two scenarios for a DAG. Imagine many blocks per second But for simplicity just say two miners. You're mining 1 and you are mining 2. We are mining 4 transactions. We both create blocks because it's real time and there's no time to coordinate and there are two scenarios. The less optimistic scenario is where you select tx 1, tx 2, and I select tx 1, tx 2 from the mempool and what happens in this case is that bcause these transactions are not conflicting, then no harm was done in the sense that both transactions were approved by the system. However you wasted space. So tx 1 and tx 2 took the space of 4 transactions. So this is just a very simple example and of course the optimistic scenario or the green scenario on your left hand side would be that you somehow selected tx 1, 2, and the other miner picked tx 3, and tx 4. So this is a good outcome.

如何充分利用DAG的全部吞吐量或至少接近DAG可以提供的全部吞吐量？我会告诉你为什么这不是一个微不足道的挑战。我们有两种DAG情况。设想每秒产生许多块，但为简单起见，只假设有两名矿工。一个是矿工1，另一个是矿工2.我们正在挖掘4笔交易。我们都创建了块，因为它是实时的，没有时间进行协调，并且有两种情况。不太乐观的情况是你选择交易1，交易2，我也从矿区中选择交易1，交易2，这种情况下会导致这些交易不会发生冲突，那么两者都没有造成任何伤害由系统批准交易。然而，你浪费了空间。因此交易1和交易2占用了四4个交易的空间。这仅仅是一个非常简单的例子，当然以你的角度，乐观一些的情况或者是绿色的情况是你以某种方式选择了交易1,交易2，而另一个矿工选择了交易3和交易4。所以这是一个很好的结果。

The main question of our research here is how do we make sure that we are closer to the green area than the red area? Assume we have a good consistency rule and a good blockDAG protocol. How can we make sure that transactions are not duplicated across chains?

我们这里研究的主要问题是我们如何确保我们更接近绿色区域？假设我们有良好的一致性规则和blockDAG协议。我们如何能确保交易不会在多条链之间重复？

If you don't do anything and let miners mine greedily the top transactions in their mempool, then the DAG gains nothing in terms of throughput. We did a simple solution generated by my colleague. The green curve is this throughput of a DAG. By using a greedy mining policy where every miner selects the top transactions from the mempool. The yellow curve represents a chain structure with the same parameters. The chain throughput and the DAG throughput are almost identical. So DAG gains us nothing in terms of throughput, but it does gain us something in terms of confirmation time. These curves are very far from optimal. We can ain much more-- DAG has more capacity if we were more wise and used a better scheme to coordinate between miners.

如果你不做任何事情而让矿工贪婪地挖掘他们内存池的手续费最高交易，那么DAG在吞吐量方面没有任何提升。我们做了一个由我的同事提出的简单解决方案。绿色曲线是DAG的吞吐量。通过使用贪婪的挖矿策略，每个矿工选择矿区的手续费最高的交易。黄色曲线表示具有相同参数的链结构。链吞吐量和DAG吞吐量几乎相同。所以DAG在吞吐量方面没有给我们带来任何好处，但它确实在确认时间方面有所提升。这些曲线远非最佳。我们可以做得更多————如果我们更明智并且使用更好的方案来协调矿工之间，DAG将有更多能力。

<img width="987" alt="2018-07-13 11 47 27" src="https://user-images.githubusercontent.com/39436379/42671455-9850b432-8692-11e8-830d-3614f888fc98.png">

So here's the good news and perhaps the main thing I want to message to convey to you. The miners are incentivized to some extent to avoid collisions and avoid tx duplication and contribute unique transactions and increase throughput. So this is a natural incentive of miners to contribute to the healthiness of the system.The reason is simple. If we collide in th same system, if we choose the same tx for our blocks, then we need to split the fee, you will get half of the fee or some other portion I will get the other portion but we can't pay the fee twice. And we both embedded the tx in our block. only one of us will get the fee or maybe we will split it but we won't be able to extract the entire fee twice. So there's incentive to coordinate and avoid collisions.

所以这是好消息，也许是我想传达给你的主要内容。矿工们在某种程度上受到激励，以避免冲突和重复，并提供独特的交易和提高吞吐量。因此，这种对矿工天然的激励有助于系统的健全。原因是简单的。如果我们在同一系统中发生冲突，如果我们为我们的区块选择相同的交易，那么我们需要平分手续费，你将收取一半的手续费，而我将在其他部分收取另一半手续费，但我们不能支付两次费用。我们都在我们的块中加入了交易。我们中只有一个人会收取费用，或者我们会平分它，但我们将无法提取全部费用两次。因此，是存在激励来协调和避免冲突的。


##Chicken game

##胆小鬼博弈

And now we're headed to a more formal treatment of this problem. I hope the chicken game is known to most of you. It's a famous game to model conflicts. You have two parties driving towards each other and heading to collision. And the first one to swerve and chicken out is the chicken and the one to stay the course is the winner and daredevil.And of course, if either you chicken out then the other party wins. And if we both dare then there's a collision. The transaction selection game in blockDAG is quite similar. We have tx1, tx2 in the mem pool and one of the transactions is more expensive (tx1) let's say it pays 2 millibitcoins and tx2 pays 50% as much. So now we have to decide how to let numbers should be different I'd say tx1, 3 millibitcoin and tx2 would be 2 millibitcoin.If we both select tx1, the most high paying transaction, we will collide and need to divide the fee between us. But if we both select tx2, we need to divide a lower fee between us. So we need to coordinate and select the transactions. This would be the optimal setup.Strategies in this game is the important point you have in game theory. Pure strategies which is simply which transaction to select, and you have mixed strategies which is using randomness to select. In a mixed strategy setup I can toss a coin and then decide which transactions to select. In game theory, using randomness you can achieve much better results for all players. In practical terms, we are looking at setup in a DAG where miners will randomize over the mempool to select transactions. In bitcoin, a miner selects the top transactions. But in a DAG, the best thing for you to do is to randomize over the mempool according to some distribution and then use this to maximize your throughput.

而现在我们正在对这个问题进行更形式化的处理。我猜大多数人知道胆小鬼博弈。这是一个模拟冲突的著名游戏。你有两队，都朝着对方行驶并即将碰撞。第一个转向并胆怯的人是胆小鬼，另一个继续行驶的人是胜利者并且是胆大妄为的。当然，如果一队退缩，那么另一队就会赢。如果两队都敢一直前进，那么就会有碰撞。blockDAG中的交易选择游戏与胆小鬼博弈非常相似。我们在内存池中有交易1和交易2，其中一个交易是贵一些的（交易1）假设它支付2毫比特币的手续费，交易2只支付交易1的50%。所以现在我们必须决定如何让数字变得不同，假设交易1支付3豪比特币的手续费，交易2支付2豪比特币的手续费。如果我们都选择交易1，支付最高手续费的交易，我们会冲突并且我们之间需要平分手续费。但是如果我们都选择了交易2，我们之间需要平分一笔更低的手续费。所以我们需要协调和选择交易。这将是最佳方案。在博弈论中, 博弈策略是非常重要的, 纯策略只是简单地选择交易, 而你也可以使用混合策略来随机选择。在混合策略方案中，我可以掷硬币，然后决定选择哪些交易。在博弈论中，使用随机性可以为所有玩家获得更好的结果。实际上，我们正在寻找DAG中的方案，矿工将通过内存池随机选择交易。在比特币中，矿工选择手续费最高的交易。但是在DAG中，最好的办法是根据某些分布随机从内存池中选择交易，然后使用它来最大化吞吐量。

How do we solve this chicken game? In game theory, there are several approaches. The top left, scenario is that there's adversarial setup. You are a mining pool, you are paranoid, you think everyone is going to sabotage your profits and they want your revenue to drop. In that scenario, your safe solution is that you just want to guarantee yourself the best response against everyone else that wants to sabotage me and lower my profits. This is quite adversarial and does not fit loving bitcoin community so we might consider more cooperative setups. The selfish setup is a more common one, which is more similar to the Nash equilibrium model where every party pursues their own interest but somehow the invisible hand brings us to equilibrium under which nobody gains from deviating. This is a common concept. There's also an interesting equilibrium concept invented by Robert Oman he's a novelist from the Hebrew University and this is coordinated equilibrium where everyone is selfish but we have signal for some coordination and in mining the signal could be the randomness from previous blocks. Perhaps we can use some randomness from previous blocks to select more wisely transactions to our mempool. And then most optimistic, everyone is playing just for the sake of the health of the system. You can think of bitcoin's early day as a setup as where the only concern of everyone was to maximize social welfare, and miners were confirming zero-fee transactions and there were less incentives back then.
block.

我们如何解决胆小鬼博弈？在博弈论中，有几个方法。左上角的方法是对抗性的方法。你是一个挖矿池，你很偏执，你认为每个人都会破坏你的利润，他们希望你的收入下降。在这种情况下，你的安全解决方案是，你只是想保证自己对其他想要破坏我并降低利润的人做出最好的回应。这是充满对抗性的，不适合和谐的比特币社区，所以我们可能会考虑更多合作性的方法。利己方法是一个更常见的方法，与纳什均衡相似，每个团体都追求自己的利益，但不知何故，无形的手把我们带到了平衡之中，没有人从违规中获益。这是一个常见的概念。罗伯特·阿曼提出了一个有趣的均衡概念，他是希伯来大学的小说家。这是个协调的均衡，每个人都是利己的，但我们有一些协调的信号，而在挖矿过程中，信号可能是先前区块的随机值。也许我们可以使用上千个区块生成的随机数来更明智地选择交易到内存池。然后最乐观的是，每个人都只是为了系统的健康而参与。你可以设想下早期比特币的情景, 每个人唯一关心的就是最大限度地提高社区福利, 并且矿工也会去确认零手续费的交易。当时的激励措施较少。

##Max social welfare

##最大化社区福利

We can visit several of these ideas but let's begin with max social welfare. This is the simplest and most optimistic. What would be the solution here? All the miners are cooperating, they just ask us, we don't know how they coordinate, there's a design mechanism undre which the DAG is utilized and we won't collide and the throughput is at maximum. The solution in this case is rather simple. The construction to the miners would be just select a transaction in random uniform distribution from your mempool, so let's say you have 6 transactions or thousands, you do this by tossing a fair coin between them until some threshold, and you accept all transactions uniformy. The threshold is what's the capacity of the DAG, you can't overcome this capacity, and it's very famous and simple result from queueing theory that says that you won't have any collisions in the system. The buckets of transactions would be sparse enough that collisions will be negligible. This setup of course is a bit naive. There's two catches.

我们可以研究其中的几个想法，但让我们从最大的社区福利开始。这是最简单和最乐观的情况，这种情况有什么解决方案？所有的矿工都是合作的，他们只是问我们，我们不知道他们如何协调，有一个基于DAG设计的机制，使我们不会碰撞，吞吐量达到最大。在这种情况下，解决方案相当简单。对矿工的建设只是从你的内存池中选择统一随机分配的交易,所以假设你有六笔交易,当然,可以有成千上万的交易，你都可以通过在它们之间抛出一个公平的硬币直到达到某个阈值来实现这一点，并且你接受所有交易统一。阈值是DAG的容量，你不能超过这个容量。这是一个非常着名且非常简单的排队理论结果，它说这实际上意味着你不会在系统中发生任何冲突。交易分组足够稀疏时碰撞可以忽略不计。当然，这个方法有点幼稚。这个方法有两个缺点。

This is strategically unstable. We're assuming that everyone is maximizing social welfare and this is not always the case. There's a more inherent problem with this approach which is that it does not allow for differential service to urgent transactions. You can't prioritize transactions by fees. So I am not able to signal to miners that I need this tx faster, I am willing to pay more. So if they are not picking transactions by fees, then everyone waits about the same time. And some transactions will never be approved. You can think of this as the highway where everyone is stuck in the same traffic. As a system designer, is that really what you want to achieve? You want a system that has express lanes, where people are willing to pay more. Assume there's a payment for the express lane, of course. They are willing to pay more to get to their destination faster. And then you will have this curve that says the larger you pay, the faster you get inside. So this is one thing you want to design the system according to.

这在战略上是不稳定的。我们假设每个人都在最大化社区福利，但事实并非如此。这种方法本身存在一个问题，即它不允许对紧急事务进行差别服务。你无法按手续费确定交易的优先顺序。因此，我不能告诉矿工，我需要更快地完成交易并且我愿意支付更多手续费。因此，如果他们不按手续费选择交易，那么每个人都会在同一时间等待。有些交易永远不会被确认。你可以将此视为每个人都处于同一交通环境的高速公路。作为系统设计师，你真正想要实现的目标是什么？你想要的系统是人们愿意支付更多手续费拥有快车道。假设快车道需要付款，他们愿意支付更多费用以更快地到达目的地。然后你会得到这条曲线，说明你支付的越多，你就越快开始完成交易。所以这是你设计的系统所要参考的一个方面。

##Other strategies

##其他策略

If you have high utilization of the DAG with collision avoidance, then you can't serve the express lane and can't have prioritized transactions. Because you need signal to miners to back off from expensive transactions so that they don't collide. And on the other extreme, if you allow for fast confirmation times, then you will inevitably suffer from collisions. So what you see here is simply the function of how long have the numbers of collisions of a transaction as a function of the of the of its waiting time.挖矿 Only transactions that allow themselves to wait a long time the buffer suffer no collisions. Transactions that want to be confirmed fast will suffer collisions and would need a higher fee.

如果你有很高的DAG利用率，并且可以避免碰撞，那么你无法提供快车道服务，也无法确定交易优先级。因为你需要通知矿工们退出高手续费的交易，这样他们才不会发生碰撞。另一个极端，如果你允许快速确认时间，那么你将不可避免地遇到冲突。*所以你在这里看到的只是与交易冲突数和其等待时间的函数有关的函数。只有允许等待很长时间的交易，在缓冲区中不发生碰撞。想要快速确认的交易将遇到碰撞，并且需要更高的手续费。

In the chicken dare situation, what happens in the real world? The problem here is intractable. Usually in these games, nash equilibrium is hard to find. But the spirit of Nash equilibrium is something in the form of if I see you cooperating, then I will cooperat. But if you are greedy and you are always picking the highest transactions for your block, then I will also be greedy and pick the highest paying transactions. I apologize for the militaristic tone, but this is tit-for-tat strategy. And it helps understand how mining pools will behave.

在胆小鬼博弈的场景下，在现实世界中会发生什么？这个问题是棘手的。通常在这些游戏中，很难找到纳什均衡。但是纳什均衡的精髓就是如果我看到你打算合作，那么我就会合作。但是如果你贪婪而且你总是为你的区块选择最高手续费的交易，那么我也会是贪婪的并选择手续费最高的交易。我为自己军国主义的语气道歉，但这是针锋相对的策略。它有助于了解矿池的行为方式。

In a world where mining pools don't hide their identity, if a mining pool really will be greedy， then other mining pools will start sabotaging that pool by colliding with their transactions. It's an interesting game theoretic framework. Chicken dare is not unique to two mining a blockDAG transactions.

在矿池不隐藏其身份的世界中，如果一个挖矿池是贪婪的，其他矿池将开始通过与贪婪挖矿池的交易碰撞来破坏该池。这是一个有趣的博弈论框架。胆小鬼博弈并不是只满足两个矿工挖掘一个blockDAG交易的情况。

We did a more modest goal-- namely, we computed the nash equilibrium for the single shot game. We only played once, what would be the nash equilibrium? The qualitative message of the nash you can see here that we picked high paying fees with high probability. So if a use wants to embed his transaction fast, he will put a large fee, but then we will all sort of collide. The curve does not reach probability 1. Even if I am paying a high fee nobody will be greedy enough to pick it with certain probability. More good news for you is that nash performs quite well. The utilization of the DAG under nash equilibrium even if everyone is selfish, it wouldn't be so bad. It's around 72% on this graph. So 70% of the graph obtains unique content and unique transactions. It's a surprising result. If everyone was selfish then we would revert back to the greedy scenario, you wouldn't be greedy even if you're selifhs because you want to randomize over your mempool to avoid collisions.

我们做了一个更适中的目标——即，我们计算了单发射击的纳什均衡。我们只玩了一次。什么是纳什均衡？你可以看到纳什均衡的定性信息，我们很有可能选择高额手续费的交易。因此，如果一个用户希望快速嵌入他的交易，他将支付大笔手续费，但随后我们有可能会发生冲突。曲线未达到概率1。即使我支付高额手续费，也没有人会贪婪地以一定的概率选择它。对你来说更好的消息是，纳什均衡的表现相当不错。在纳什均衡下使用DAG，即使每个人都是自私的，也不会那么糟糕。在这张图上，它大概是72％。因此，图表中显示70%能获得唯一的内容和唯一的交易。这是一个令人惊讶的结果。如果每个人都是自私的，那么我们会回到贪婪的方法，即使你是自私的也不会贪婪，因为你想随机化你的内存池以避免碰撞。

<img width="1163" alt="2018-07-13 12 00 38" src="https://user-images.githubusercontent.com/39436379/42671781-7dc06fe8-8694-11e8-9185-d6a625d742fd.png">

##Quality of service

##服务质量

Can we improve utilization a bit more? Another interesting graph we must talk about is the quality of service level. As I said, the problem with the maximum social welfare solution is that you can't prioritize transactions. Here the tall grey bars represent max social welfare strategy. Everyone waits the same time. And then the blue bars represent the nash equilibrium where you approve less transactions- only those that have 3 millibitcoin and those exact numbers do not matter of course, it's a dummy model. As you increase the fee, you wait less. We want this feature. We want high-paying transactions to wait less. And we have other strategies like exponential strategy. If everyone is greedy, then you would only approve a small set of transactions. This is a good illustration of the challenge.

我们可以提高一点利用率吗？我们必须讨论的另一个有趣的图表是服务质量水平。正如我所说，最大的社区福利解决方案的问题是，你不能划分交易的优先级。这里高的灰色条形代表了最大的社区福利策略。每个人都在等待同一时间。然后蓝色条代表纳什均衡，你批准的交易较少——只有那些有3毫比特币的交易，那些确切的数字当然无关紧要，它是一个虚拟模型。随着你增加手续费，你等待的时间更少。我们想要这个特征。我们希望交易手续费增加而等待时间减少。我们还有其他策略，如指数策略。如果每个人都是贪婪的，那么你只会批准一小部分交易。这是挑战的一个很好的例证。

<img width="1117" alt="2018-07-13 1 11 12" src="https://user-images.githubusercontent.com/39436379/42673489-45045aa2-869e-11e8-9e29-c01bd8a327d1.png">

##Correlated equilibrium

##相关均衡

Perhaps with coordination between miners we can achive better results than nash. We could use previous block's randomness. Preliminary results show that we are able to utilize the DAG better. Needs further investigation.

也许通过矿工之间的协调，我们可以获得比纳什均衡更好的结果。我们可以使用前一个块的随机值。初步结果表明我们能够更好地利用DAG。需要进一步调查。

##Scaling and incentives

##扩展和激励

In bitcoin selfish mining is a known attack and deviation on the protocol. It's risky, it only works in a long term after two
weeks at least. And so arguably the reason why we didn't see selfish mining until now or at least not in notable amounts was that playing with your mining node just to withhold your blocks and risk losing it all is too risky and miners wont engage in such behaior. Unforotunately in a DAG, mining from the mining protocol is much more available to you, you just need to select transactions in a certain way or hold your blocks for a few seconds. So if I am withholding my block for 2 bloks, that's not going to hurt me. It's a much more granular decision for how to optimize my mining, and the risks for the miner are less. The harm to the system is relatively marginal. Also lazy selfish mining is another concern- it might not have communication infrastructure either deliberately or by laziness and doesn't propagate blocks to the network. This pool might not lose too much profit and other miners might be harmed, this is an interesting topic for us so we're investigating it.

在比特币中，自私的挖矿是一种已知的攻击和对协议的偏离。它有风险，它至少在两周后才能长期使用。因此可以说，为什么我们直到现在才看到自私的挖矿，或者至少没有显着的数量，因为挖矿节点只是为了扣留你的区块而冒着失去一切的风险太大，矿工们不会参与这样的行为。不幸运是，在DAG中，从挖矿协议中挖矿更加适合你，你只需要以某种方式选择交易或者将块扣留几秒钟。因此，如果我扣留了我的块中2个，那不会对我造成损失。对于如何优化我的挖矿协议来说，这是一个更为细化的决策，并且矿工的风险更小。对系统的危害相对较小。懒惰的自私挖矿也是另一个问题——它可能是故意或因低速的通信设备，没有及时将块传播到网络。这个池可能不会损失太多的利润而其他矿工可能会受到伤害，这对我们来说是一个有趣的话题，所以我们正在研究它。

When implementing blockDAG protocols, the incentives really matter. Bitcoin has 1 block every 10 minutes. Things are are pretty robust at least in the mining at least as you stay in the same chain but in DAG things are very sensitive.

在实施blockDAG协议时，激励措施确实很重要。比特币每10分钟产生1个块。至少在挖矿中至少当你留在同一个链条但在DAG时，事情是非常敏感的。至少在你停留在同一个链中挖矿，激励措施是非常强大的，但在DAG中，激励措施是非常敏感的。
