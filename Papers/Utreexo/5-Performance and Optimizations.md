> Source: https://eprint.iacr.org/2019/611.pdf
# 5 Performance and Optimizations

# 5 性能和优化

## 5.1 Reducing proof length

## 5.1 减少证明长度

While a single inclusion proof will be of log(n) size, we can send multiple simultaneous proofs in less space than sending each proof individually.
The size reduction depends on the proximity of the leaves to be proven.
Here are the two extreme examples: if two leaves are in diﬀerent trees, there is no overlap in their proofs and two full-sized proofs are needed.
If, however, the leaves are adjacent, then no additional data is needed and the second inclusion proof is obtained for free.
When aggregating proofs, we send a sparse forest, covering every leaf to be proven, rather than a branch.
Proof branches are provided only up to the point where they intersect an already provided branch.
We can save even more space by omitting hashes that will be computed by the veriﬁer; parent hashes can be omitted if both child hashes are contained in the proof.
In the extreme case, the proof for all the leaves in a tree is just the leaves themselves, as all intermediate hashes can be computed from the leaves.

尽管一份包含证明的大小为log(n)，但比起每次只发送一份证明，
同时发送多份证明可以节省更多的空间。
能节省多少空间取决于要证明的叶子的接近程度。
这里有两个极端的例子：
如果两个叶子在不同的树中，则它们的证明不会重叠，因此需要两个完整的证明。
但是，如果叶子相邻，则只需要一份证明，不需额外开销即可获得第二份证明。
同时发送多份证明时，我们只需发送一个稀疏的森林，
而不是有多少个叶子要证明就要发送多少个证明分支。
这个森林虽然稀疏，但足以覆盖所有要证明的叶子。
提供一个证明分支时，
只需要提供从叶子到该分支与其它已提供的分支相交的部分。
通过省略校验器会生成的哈希，我们可以节省更多空间。
如果两个子哈希都包含在证明中，则可以省略父哈希。
在极端情况下，树中所有叶子的证明都仅仅是叶子本身，
因为所有中间哈希值都可以通过叶子计算出来。

In Bitcoin, transactions are aggregated into blocks in order to be conﬁrmed via proof-of-work.
When compact state nodes propagate blocks, the existence of all UTXOs spent in the block (every input in the block) can be proven with a sparse forest.
In cases where single transactions are being sent, a sparse forest proving all inputs within a transaction can be sent along with the transaction.
To save bandwidth at the cost of some memory and complexity, clients can maintain their own sparse forest in memory.
When sending a transaction, nodes send only the positions of the UTXOs consumed.
Receiving clients then respond with a proof request, indicating which nodes in the forest are needed to get up to an intersection with nodes it already has in memory.

在比特币中，交易被汇总为区块，以便通过工作量证明进行确认。
当紧凑状态节点传播区块时，
可以用稀疏森林证明在区块中花费的所有UTXO（区块里的每个输入）的有效性。
在发送单笔交易的情况下，可以将证明交易内所有输入的稀疏森林与该交易一起发送。
客户端可以在内存中维护自己的稀疏森林，这样虽然会占用一些内存，
并加大了计算的复杂性，但却可以节省带宽。
发送交易时，节点只需要发送被花费的UTXO的位置。
客户端接收到交易后以含有证明的请求进行响应，该请求会告诉对方，
还需要哪些森林里的节点才能与客户端内存中已经存在的森林节点相交，
从而形成一份完整的证明。

## 5.2 Short trees

## 5.2 矮树

As the trees are of diﬀerent heights, the length of an inclusion proof is dependent on the tree in which the leaf to be proven resides.
Bitcoin’s UTXO set exhibits a power-law distribution of UTXO durations, or the length of time between when a UTXO is created and when it is deleted.
In fact, many UTXOs are created and destroyed within the same block, in which case we can ignore them as they have no eﬀect on the accumulator.
Among UTXOs that do persist, many only last a few blocks, while some last for hundreds of thousands.
As we insert leaves into the forest on the right side, they will start oﬀ in the smaller trees, with shorter inclusion proofs.
UTXOs which have made their way to the larger trees on the left tend to have been around for thousands of blocks, and proofs in the larger trees will be less frequent.
The short-duration UTXOs will be added into short trees to the right and will be removed using smaller proofs, often with many adjacent leaves being deleted in the same block, which further reduces proof sizes.

由于树的高度不同，所以包含证明的长度取决于要证明的叶子所在的树。
在比特币的UTXO集里，
一个UTXO的持续时间（即UTXO从被创建到被删除之间的存活时间）服从幂次法则。
实际上，很多UTXO的创建和销毁都发生在同一个区块里。
在这种情况下，我们可以忽略这些UTXO，因为它们对累加器没有影响。
而在那些跨越区块存活的UTXO当中，有很多UTXO只会跨越数个区块存活，而有些则会跨越数十万个。
当我们将叶子插入森林的右侧时，叶子一开始会进入较小的树，
这时它们对应的包含证明就比较短。
进入左侧较大的树里的UTXO往往已跨越数千个区块一直存活着，
而较大的树里的证明被使用的频率没有那么高。
短期存活的UTXO会被添加到右侧的矮树中，
并且这些UTXO被删除时需要使用的证明也比较小。
它们通常是和同一区块里许多相邻的叶子一起被删除，这就进一步减小了证明的大小。

## 5.3 Forest Caching Space/Space Trade-oﬀ

## 5.3 森林的缓存空间以及空间的权衡

A node implementing the Utreexo accumulator can fully verify all inclusion proofs by storing only the roots of each tree in the forest, which on average is log(n)/2 hashes.
The additional size of the inclusion proofs can be significant when initially synchronizing to the blockchain – if proofs are naively added to every transaction input, their size exceeds that of the transactions themselves.

实现Utreexo累加器的节点可以只存储森林中每棵树的根，
从而来全面验证所有的包含证明。
平均下来，需要验证的哈希数目是log(n)/2个。
最初与区块链同步时，包含证明会占用大量的额外空间。
这是因为如果简单地将证明添加到每笔交易的输入中，
证明的大小就会超过交易本身的大小。

The simplest way to reduce proof sizes is to eliminate redundant data among multiple proofs.
Sending a sparse forest which simultaneously proves all inputs within a block, as described above, gives us the ﬁrst signiﬁcant space reduction.

减小证明尺寸的最简单方法是消除多个证明中的冗余数据。
就像上文描述的那样，我们可以发送一个稀疏森林，
用它一次性证明一个区块中的所有输入，
这样就可以显著减少空间占用。这是第一个优化手段。

Further proof size reductions can be achieved due to the redundancy of temporally rather than spatially distinct proofs.
As noted above, many UTXOs persist only for a short duration.
In the case where a UTXO persists for only a single block, the prover would be sending data that was known to (in fact computed by) the veriﬁer just a single block prior.
(In IBD scenarios, this could be a fraction of a second.) If the veriﬁer had kept all the hash tree information they computed from the last block, they would not need proofs for these short-lived UTXOs.

除了消除不同的证明在空间上的冗余以外，
另一个减小证明大小的方法是消除时间上的冗余。
上文说过，很多UTXO的存活时间都很短。
如果一个UTXO只在一个区块里存活的话，
证明方会发送验证方所知道的（实际上是验证方计算出的）一个区块之前的数据。 
（在IBD【译注：Initial Block Download，即初始区块下载】的情况下，这可能不到一秒钟。）
如果验证方能够保存上一个区块计算出的所有哈希树信息，
那就不需要这些存活时间短暂的UTXO的证明。

While the idea of keeping extra data from the forest (which takes up space) seems counter to the point of using an accumulator (to take up less space), there are good reasons to do this.
Without the ability to store and
remember parts of the forest, using an accumulator is all-or-nothing where the choices are to either run a traditional node with a database storing the entire UTXO set or an accumulator node with only a few hashes for tree roots.
With the ability to store parts of the forest, this choice becomes a gradient, allowing users to select the memory vs.
bandwidth trade-oﬀ they wish to make.

尽管将森林中（占用空间）的数据额外保存下来的想法似乎与使用累加器（以占用更少空间）的初衷相互矛盾，
但是这样做有充分的理由。
如果我们不存下森林的一部分，
那么使用累加器就是一个非此即彼的选择：
要么选择传统的全节点，在数据库里存下整个UTXO集的数据库；
要么选择累加器节点，只存储少量的树根哈希值。
有了存储部分森林的功能，这种选择就会变得更柔和，
用户可以在内存与带宽的权衡之间取得自己期望的折衷方案。

![2](./Figures/2.png)
> Figure 2: Distribution of UTXO duration in the Bitcoin blockchain.
Short lifetimes are very common, and durations of 0 blocks even more so.
0 duration UTXOs are not displayed as they do not aﬀect the accumulator (and don’t ﬁt on a log-scale x-axis).
Note the distinct peaks at block durations of 6, 100, 433, and 1000.
The 100 block delay for coinbase TXOs is a consensus rule, while the other peaks are likely due to user preferences and behavior.

> 图2：比特币区块链中UTXO持续时间的分布。
UTXO的寿命普遍非常短，0区块的寿命更是常见。
图中没有显示持续时间为0的UTXO，
因为它们不会对累加器产生影响（并且它们也不适用对数刻度的x轴）。
请注意持续时间【译注：即x轴】分别为6、100、433和1000对应的四个不同的峰值。
x轴100处出现峰值是因为在共识规则中，coinbase TXO的延迟是100个区块。
而其他峰值可能是由用户的偏好和行为而引起的。

At one extreme of this gradient, nodes minimize storage and memory requirements, keeping only the roots of the hash trees, which never exceed a kilobyte.
At the other extreme, nodes cache the entire hash forest and do not need downloaded proofs at all, just like a node with the full UTXO set (this is in fact a bridge node).
Between these two extremes, there is a gradual trade-oﬀ between network traﬃc and in-ram or on-disk storage.
The more of the forest a node caches, the smaller the proofs it will need to download.
Fortunately, due to the fact that transaction data exhibits a power-law distribution in the duration of UTXOs, this trade-oﬀ is not linear.
The ﬁrst few megabytes of caching space give a large reduction in proof download size, after which there are diminishing returns as more memory is devoted to forest caching.
For the amounts of memory we would expect to see in low cost, widely used computing devices, the download overhead for IBD is quite reasonable.

一种极端情况是，节点将存储和内存占用降到最低，
仅保留哈希树的根，而根的内存占用永远不会超过千字节。
另一种阶段情况是，节点缓存整个哈希森林，根本不需要使用从网络下载的证明，
就和完整存储了整个UTXO集的全节点（实际上这是一个桥节点）一样。
在这两种极端之间，网络流量与内存或磁盘的空间占用之间存在着渐变的折衷权衡。
节点缓存的森林数据越多，需要下载的证明就越小。
幸运的是，由于UTXO所代表的交易数据的存活时间遵循幂次法则，因此这种折衷不是线性的。
最初只要占用几兆的缓存空间就可以大大减少证明的下载量，
但之后这个收益就会放缓，因为需要投入更多的内存用于缓存森林。
在我们常见的广泛使用的低成本计算设备上，IBD的下载开销可以减少到非常合理的程度。

![3](./Figures/3.png)
> Figure 3: Fractional distribution of UTXO duration in the Bitcoin blockchain.
This is another view of the same data from Figure 2.
From this plot we can see that 40% of UTXOs last for 20 blocks or less, suggesting that a look-ahead cache of 20 blocks can reduce proof sizes by approximately 40%.

> 图3：比特币区块链中UTXO持续时间的分数分布。
这是图2的相同数据的另一个视图。
从该图中可以看到40％的UTXO只会跨越20个甚至更少的区块，
这表明只要往前缓存20个区块就可以减少大约40％的证明大小。

Another advantage of storing partial forest data is that non-root data is easily recoverable.
The partial forest data can be stored in volatile memory and not copied to persistent disk, with only the forest roots being maintained on disk.
A node can forget all forest data except for the accumulator roots
when it shuts down, and resume synchronization later.
The node will need to download additional data (the hashes forgotten) but will not have to backtrack in the synchronization process.

存储部分森林数据的另一个优点是非根数据可以很容易被恢复。
部分森林数据可以存储在易失性内存中，
而不必持久化到磁盘上，只有森林里的根才需要存到磁盘上。
节点关机的时候可以丢弃累加器的根以外的所有森林数据，
等之后同步的时候再恢复那些数据。
同步的时候，节点需要下载额外的数据（也就是那些被丢弃的哈希），但不需要回溯。

## 5.4 What to cache

## 5.4 缓存什么

When a client is performing IBD and decides to cache some portion of the forest data, what should they retain, and what should they forget? A simple, but still fairly eﬀective strategy would be to remember all leaves added to the forest until either they are removed, or some number of blocks (the look-back period) have passed, at which point they are forgotten.
Given the distribution of TXO lifetimes, a small look-back period can signiﬁcantly reduce the sizes of proofs.

当客户端执行IBD并决定缓存部分森林数据时，应该缓存什么，丢弃什么呢？
一种简单但有效的策略是记住添加到森林中的所有叶子，
直到它们被删除或者时间上已经过去了一定数量的区块（回溯期），
那时叶子就会被丢弃。
给定TXO生命周期的分布，
只需要一个小的回溯期就可以显著减小证明的大小。

Look-back caching is sub-optimal, however, as it would store many nodes for a period and then forgot them before they are used.
The space these nodes took up was wasted, as they displaced other nodes which could have potentially reduced proof sizes.
While this problem is generally unavoidable in caching algorithms, we can prevent this from happening completely in the case of IBD.
The server which stores the blocks and inclusion proofs is already fully synchronized, and for every TXO created, it knows in which block (if any) that TXO is consumed.
(For TXOs still in the UTXO set, the server considers that block to be “never”.) Servers can send this time to live (TTL) value to the synchronizing client as a hint, allowing the client to cache look-ahead nodes rather than look-behind nodes.
This also removes the need for the client to keep track of the insertion time of the cached leaves, as leaves are eliminated from memory only when the TXO is spent, and never from cache eviction.

但是，回溯缓存策略不是最佳的选择，
因为节点【译注：这段话说的节点都是指森林里的哈希节点】可能会被缓存上好一段时间，
却还没得到使用就会被移出缓存。
这些节点会浪费缓存空间，因为它们占用了其他节点的空间，
而那些节点可能可以减小证明的大小。
尽管在缓存算法中通常无法避免这种问题，
但在IBD的情况下，我们是可以完全避免这种情况发生的。
IBD的时候，存储区块和包含证明的服务器已经完全同步，
对于每个被创建的TXO，
它都知道这个TXO是在哪个区块里被花费的（如果这样的区块存在的话）。
（对于仍处于UTXO集中的TXO，服务器认为这样的区块“从未存在”。）
因此服务器可以将TXO的生存时间（TTL）值作为提示发送给正在同步的客户端，
从而允许客户端提前有针对性地缓存节点，
而不是回溯历史，缓存所有已经被添加的节点。
这样客户端就不用记录缓存里每个叶子的插入时间，
叶子只有在TXO真正被花费时才会从内存中被删除，
而不会仅仅因为缓存回收就被移出内存。

The look-ahead caching is dependent on the cooperation of the IBD server.
While this server provides a hint of the UTXO’s TTL, the server does not provide any cryptographic proof for it.
Such a proof could be constructed using SPV proofs of transactions spending the TXO.
Unfortunately the size of such proofs would oﬀset all download savings from the look-ahead, defeating the purpose of sending them.
While the IBD node is “trusted” to give TXO TTL hints, this trust is only for an optimization, and the worst a malicious IBD server can do is cause the client to cache leaves ineﬃciently, resulting in higher network transfer sizes (which the server also suﬀers itself).
A client could perform spot checks on the hints the server provides and disconnect from a server providing erroneous information.

提前缓存的效果取决于IBD服务器的协作程度。
虽然IBD服务器提供了UTXO TTL的提示，
但该服务器并不会为其提供任何密码学证明。
当然，可以使用花费TXO的交易的SPV证明来构造这样的证明。
但是此类证明的数据量会抵消提前缓存所省下的所有下载流量，
违背发送这些证明的初衷。
不过，尽管我们选择“信任”IBD节点会提供提供TXO TTL提示，
但是这种信任仅仅是为了优化。
恶意IBD服务器可以做的最坏的事情不过是导致客户端无法有效地缓存叶子，
从而导致网络传输流量变大（这种情况下服务器自己的流量也会变大）。
客户端可以对服务器提供的提示执行抽查，
并与提供错误信息的服务器断开连接。

IBD presents an unusual optimization scenario in that not only are TXO TTLs known ahead of time, but TXO arrivals are known before they happen as well.
In fact, the entire sequence of TXO insertion and removal is fully known at the outset of the IBD process.
Thus for a given amount of memory there is a deterministic, optimal caching schedule which can be precomputed.
The caching schedule can be represented as a single bit for every TXO, with 0 meaning “immediately forget” and 1 meaning “remember until spent”.
We anticipate that the optimal selection algorithm in [10] can be eﬀectively applied to reduce the proof sizes for IBD beyond the reductions provided by look-ahead caching.
These caching schedules will be of reasonable size (on the order of 100MB) and are likely amenable to standard compression algorithms, unlike most of the data we deal with here, such as hashes and signatures.

IBD的优化场景是非常特殊的，
因为在IBD的情形下，我们不仅可以提前知道TXO TTL，
还可以提前预知哪些TXO会到达。
实际上，在IBD过程开始时，我们就完全知道TXO的插入和移除的整个过程。
因此，可以在有限的内存里预先算出明确的优化的缓存方案。
缓存方案可以表示为每个TXO对应一个比特，
其中0表示“立即忘记”，而1表示“记住直到用完”。
我们相信可以利用[10]中的最佳选择算法有效地减少IBD的证明大小，
该策略比提前缓存策略更能减小证明的尺寸。
这些缓存方案的空间大小都比较合理（大约100MB），
并且可能适用于标准压缩算法，
这与本文涉及的大多数数据（例如哈希和签名）不同。

We have left implementation of the clairvoyant cache scheduling for later work.
We have implemented the simpler look-ahead caching to measure network traﬃc requirements for IBD, as detailed below.
While implementing this caching strategy we observe that a ﬁxed block look-ahead, while better than look-behind, is sub-optimal, and thus there may still be room for signiﬁcant space savings.
One easily observed deﬁciency is the highly variable amount of memory used.
Client machines generally have a ﬁxed amount of memory to be used for running Bitcoin, which would ideally be full, or close to full, at all times.
In the early blocks of Bitcoin’s blockchain, there are few transactions, and many blocks with no TXOs consumed at all.
Thus clients with, for example, 100MB to dedicated to caching are using only a small fraction of that in the ﬁrst half of the IBD process.
Even in later blocks with higher transaction rates, there is still signiﬁcant variability in memory usage for a ﬁxed look-ahead strategy.

我们会将千里眼（clairvoyant）缓存调度的实现留到以后的工作中。
目前已经实现了更简单的提前缓存策略来衡量IBD的网络流量需求，如下所述。
在实施这种缓存策略时，我们观察到，
虽然提前缓存一定数目的区块固然比往后回溯的策略好，
但依然不是最优解，因此在内存空间上可能仍有很大的优化空间。
一个容易观察到的不足是使用的内存量变化很大。
客户端计算机可用于运行比特币的内存容量通常是固定的。
在理想情况下，内存始终会被占满，或接近被占满。
在比特币区块链的早期区块中，交易很少，许多区块根本没有花费TXO。
因此，假如一个客户端将100MB用于缓存的话，
那么IBD过程的前半段就只会用到一小部分缓存。
即便是在后面交易频率更高的区块中，
采用提前量固定的缓存策略所占用的内存空间仍然很不稳定。

## 5.5 Measuring Performance

## 5.5 性能测试

We implemented a Utreexo library and IBD simulator.
This simulator iterates through the Bitcoin blockchain up to block 546000, adding and removing TXOs from the accumulator.
We used blake2b as our hash function, and wrote the implementation in Go, compiled with go1.10.4 linux/amd64.
Simulations were run on a machine with an AMD Ryzen 7 1700 processor and 32GB of RAM, running Ubuntu 18.04.
Our implementation is publicly available at [11].

我们实现了一个Utreexo库和IBD模拟器。
该模拟器会遍历整个比特币区块链，直到区块546000，
并一边在累加器中添加和删除TXO。
我们使用blake2b作为哈希函数，用Go编写代码，用go1.10.4 linux / amd64编译。
模拟测试是在装有AMD Ryzen 7 1700处理器和32GB RAM，Ubuntu 18.04的计算机上运行的。
我们的实现在[11]中公开可用。

Given the power-law-like distribution of UTXO TTLs, we would expect a similar curve for download size as cache sizes increase, and our observed performance is in keeping with this expectation.
We measured peak memory usage for the entire program, and had a minimum memory usage of approximately 80MB (likely due to database and other runtime memory usage).
Due to this ﬁxed overhead, very small lookahead values of 1, 3, or 10 blocks do not seem useful.
Given that memory usage will be at least 80MB (and likely more for a real, rather than simulated node) saving a few megabytes of cache memory is undetectable, while the increase in download size is signiﬁcant.

考虑到UTXO TTL的幂次法则分布，随着缓存大小的增加，
我们可以预期下载流量也会出现类似的曲线，
而实验观察到的性能也确实与此预期保持一致。
我们测量了整个程序的峰值内存使用量，
期间最小内存使用量约为80MB（可能是由于数据库和其他运行时内存使用量所致）。
由于这个开销是固定的，因此缓存提前量小至1、3或10个区块的话，似乎没什么效果。
在内存使用量至少为80MB（非模拟的实际服务器节点可能会占用更多内存）的情况下，
仅仅节省几兆字节的缓存对人来说在内存上没省多少，但下载流量的增加却非常明显。

![4](./Figures/4.png)
> Figure 4: IBD proof size vs cache size.
Labels on the points are the number of look-ahead blocks.

> 图4：IBD证明大小与缓存大小之间的关系。蓝点上的标签是提前缓存的区块数量。

As we increase cache sizes, we see diminishing returns, and it takes nearly 12GB of memory to completely eliminate all proof downloads.
With settings of this size the Utreexo design might seem superﬂuous as well – no proofs are ever given, and the client is storing the entire forest, which is larger than the standard UTXO set database.
There are still advantages however, in that this node keeps the forest in volatile memory, only writing the tree roots to disk.
There may be machine conﬁgurations with large amounts of volatile memory but limited non-volatile storage I/O.

可以看到缓存增大，收益会递减，并且完全消除所有证明的下载流量需要大约12GB的内存。
如果要使用这么多内存，Utreexo的设计似乎也很多余——因为节点不需要网络提供任何证明，
并且客户端存下了整个森林，这个森林比标准的UTXO集数据库还要大。
不过这么做仍然有一些优点，
因为节点只是将森林存储在在易失性内存中，写入磁盘的只有树根。
有的机器可能配置有容量巨大的易失性内存，却只有有限的非易失性存储I/O。

With moderate amounts of memory, caching oﬀers signiﬁcant improvements for lower-end hardware with mechanical hard drives.
A low-cost laptop with 4GB of RAM and a 500GB mechanical drive can use a lookahead value of 1000 blocks, which uses 234MB of RAM and gives an IBD download overhead of about 33%.
We hope to bring this overhead down in the future with improved caching techniques.

对于装备机械硬盘驱动器的低端硬件而言，
只要提供适量的内存空间，就可以通过缓存获得明显的性能改进。
具有4GB RAM和500GB机械驱动器的低成本笔记本电脑可以将提前量设为1000个区块，
这需要234MB的RAM，对应33%的IBD下载开销。
我们希望将来通过改进缓存技术来进一步减少这个开销。

## 5.6 Hardening against collision attacks

## 5.6 抵抗碰撞攻击

In addition to better caching techniques, another promising method to reduce proof sizes is to reduce the length of individual hash outputs.
At ﬁrst glance, truncating hashes may seem unsafe, but Bitcoin oﬀers a unique environment which can help mitigate collision attacks on reduced length hashes.

除了改进缓存技术外，
还有一种可以有效减少证明大小的方法是缩短单个哈希输出的长度。
乍一看，截断的哈希似乎是不安全的，
但是比特币提供了一个独特的环境，
可以在哈希长度缩短后帮助抵抗碰撞攻击。

As an attacker (especially a miner) has signiﬁcant control over the accumulator’s Merkle forest, one might expect that a collision-resistant hash function is required to prevent the attacker from creating invalid proofs (inclusion proofs for elements not previously added).
In Bitcoin’s case, however, we can make collision attacks infeasible, such that an attacker would instead need to perform a second preimage attack.

由于攻击者（尤其是矿工）对累加器的默克尔森林拥有很大的控制权，
因此人们可能觉得需要一种抗碰撞的哈希函数来防止攻击者创建无效的证明（未添加元素的包含证明）。
但是，在比特币的场景下，我们可以使碰撞攻击变得不可行，
从而使攻击者被迫执行第二次原像攻击。
【译注：原像攻击是指给定一个哈希h，找到一条消息m使得hash(m) = m。
第二次原像攻击是指给定一个固定的消息m1，找到一个不同的消息m2，
使得hash(m2) = hash(m1)。】

We assume an attacker who also is able to mine a block, and thus inﬂuence a number of leaf insertions, their positions, and the data they contain.
The simplest attack would be to create TXOs txo and txo' , where h(txo) = h(txo' ).
txo is an output from a valid transaction which all nodes on the network will conﬁrm, while txo' is a made-up output of a million bitcoins that is not part of any valid transaction.
The attacker can then provide an inclusion proof for txo' , and spend the million bitcoins even though only txo has been inserted into the accumulator.

我们假设攻击者也能够挖出一个区块，
从而影响大量叶子的插入、它们的位置以及其中包含的数据。
最简单的攻击是创建TXO txo和txo'，其中h(txo) = h(txo')。
txo是有效交易的输出，网络上的所有节点都将确认该交易，
而txo'是一百万个比特币的虚构输出，不属于任何有效交易。
然后，攻击者可以提供txo'的包含证明，
并且即使只有txo被插入累加器，也可以花费那一百万个比特币。

As the attacker is able to freely create both txo and txo' , the attacker can mount a collision attack, which takes on the order of 2<sup>n/2</sup> computations, where n is the bit-length of the hash output.
If we can restrict the attacker’s ability to create either side of the collision (the hash being inserted or the hash being falsely proven) this attack is no longer feasable.

由于攻击者可以自由创建txo和txo'，
因此攻击者发起的碰撞攻击大约需要进行2<sup>n/2</sup>次计算，
其中n是哈希输出的位长。
如果我们可以限制攻击者在任意一侧（被插入的哈希或提供错误证明的哈希）创建碰撞的能力，
那么这种攻击就不再可行。

To prevent such an attack, we require that the data inserted into the accumulator be not just the hash of a TXO, which is controllable by the attacker, but instead the concatenation of the TXO data with the block hash in which the TXO is conﬁrmed.
The attacker does not know the block hash before the TXO is conﬁrmed, and it is not alterable by the attacker after conﬁrmation (without signiﬁcant cost).
Veriﬁers, when inserting into the accumulator, perform this concatenation themselves after checking the proof of work of the block.
Inclusion proofs contain this block hash data so that the leaf hash value can be correctly computed.

为防止此类攻击，
我们要求插入到累加器中的数据不仅是攻击者可以控制的TXO哈希，
还应是TXO数据与确认TXO的区块哈希的拼接。
攻击者在TXO被确认之前不知道区块哈希，并且在确认之后，
攻击者无法更改（在不花费巨大成本的前提下）该哈希。
验证者在插入累加器时，在检查区块的工作量证明之后自行执行此拼接。
包含证明会包含此区块哈希数据，以便可以正确计算叶子的哈希值。

This additional data thwarts collision attacks as the attacker needs to ﬁnd a block (which currently takes more than 2<sup>70</sup> hash operations) to create a single txo.
txo' can still be iterated through rapidly, as the attacker can use any previously computed block hash in their proof of txo' .
The number of operations required to mount a collision attack when attempts on one side are more diﬃcult can be computed by

由于攻击者需要找到一个区块（当前需要执行2<sup>70</sup>多个哈希操作）来创建一个txo，
所以这些额外的数据阻止了碰撞攻击。
当然，攻击者仍然可以快速多次尝试创建txo'，
因为攻击者可以在txo'的证明中使用任何先前计算的区块哈希。
不过当其中一侧的尝试变得更加困难时，发起碰撞攻击所需的操作次数就会变为：

<div align="center">
<img src="https://latex.codecogs.com/svg.latex?s%3D2%5E%7B%5Cfrac%7Bd&plus;n%7D%7B2%7D%7D" title="s = 2^\frac{d+n}{2}" />
</div>

where d is the diﬃculty exponent, n is the size of the hash output in bits, and s is the resulting security against collisions.
For collision attacks on 256-bit hashes, with 2<sup>70</sup> work required on one side, this would give 2<sup>163</sup> operations for a collision.

其中d是难度系数，n是哈希输出的大小（以位为单位），s是由此产生的防碰撞安全性。
如果一个256位的哈希进行碰撞攻击，
并且其中一侧需要进行2<sup>70</sup>次尝试，
那就需要进行2<sup>163</sup>【译注：即2<sup>(256+70)/2</sup>】次碰撞操作。

A second preimage attack, where txo is ﬁxed and txo' alone can be iterated through, would seem to need 2<sup>256</sup> operations to succeed.
However, the attacker doesn’t need to collide with txo, but can in fact collide with any leaf present in the accumulator.
This means the attack gets easier as the accumulator becomes larger; for 2<sup>32</sup> elements, the attack takes 2<sup>192</sup> attempts.

第二次原像攻击（txo固定，只能更改txo'）似乎需要2<sup>256</sup>次操作才能成功。
但是，实际上攻击者并不需要和txo碰撞，
而是可以与累加器中存在的任何叶子碰撞。
这意味着随着累加器的增大，攻击会变得更加容易。
对于2<sup>32</sup>个元素，攻击需要2<sup>192</sup>次尝试。

The security gain from mitigating collision attacks can be used to decrease the proof size by truncating the output length of all hashes computed.
For a 2<sup>128</sup> security parameter, and an anticipated UTXO set size of 2<sup>32</sup> or fewer, we estimate that hash outputs of 186 bits would suﬃce.
This would result in a 27% reduction in proof size with minimal complexity.
However, if the UTXO set increases, security could be degraded as second preimage attacks become easier to mount.
Once hashes are truncated, it’s not possible to retroactively increase the output size, and instead the accumulator would need to be rebuilt from scratch with larger hash outputs.
Additionally, the proof-of-work required to create a block in Bitcoin can decrease, which would also erode the protection from collisions the block hash provides.

碰撞攻击减轻之后，
我们在生成哈希时就可以缩短哈希的输出长度，
从而减少证明的大小。
如果安全参数为2<sup>128</sup>，
并且预期的UTXO集大小为2<sup>32</sup>或更小，
那么186位的哈希输出应该就足够了。
这样可以将证明大小减少27％，复杂度是最小的。
但是如果UTXO集变大，
那么由于第二次原像攻击变得更容易发动，
安全性可能就会降低。
一旦哈希被截断，那就无法逆向恢复输出的大小，
取而代之的是需要从头开始使用更大的哈希输出来重建累加器。
此外，在比特币中创建区块所需的工作量证明会减少，
这也会削弱区块哈希提供的碰撞保护。

Further protection from untargeted second preimage attacks may be gained by also committing to the leaf’s position in the leaf data.
In this case the leaves would be of the form h(txo||blockHash||position), where position is the integer index of the leaf position.
This would require the attacker to select a single leaf to target for colliding instead of allowing the attacker to collide with any leaf in the forest.
However, this technique is not applicable to our construction as leaves within the forest move due to deletions, and thus the position data salted into the hash will generally diﬀer from the leaf position when the leaf is removed.
The complexity of tracking leaf movements seems to overwhelm any savings from this technique, but we mention this idea as a diﬀerent accumulator construction, possibly closer to that in [8] may allow for ﬁxed leaf positions and shorter proofs while still being secure.

还可以通过确保叶子在叶子数据中的位置来增强对未定向二次原像攻击的防护。
在这种情况下，叶子采用h(txo || blockHash || position)的形式，
其中position是叶子位置的整数索引。
这可以使攻击者被迫选择一个叶子作为碰撞目标，
而不是允许攻击者与森林中的任何叶子碰撞。
但是，该技术不适用于本文，
因为森林中的叶子会由于删除操作而移动，
反映到哈希中的位置数据通常会与删除操作发生前的叶子位置不同。
而跟踪叶子移动的复杂性很高，可能会使本文的技术无法节省任何开销。
不过我们可以把这种想法当作一种不同的累加器。
这种思路可能更接近[8]中的设计，
[8]的设计可以固定叶子位置并缩短证明长度，同时仍保持安全性。

# Reference
[8] Leonid Reyzin and Sophia Yakoubov. Eﬃcient asynchronous accumulators for distributed pki. Cryptology ePrint Archive, Report 2015/718, 2015. https://eprint.iacr.org/2015/718.

[10] Laszlo A. Belady. A study of replacement algorithms for a virtual-storage computer. IBM Systems journal, 5(2):78–101, 1966.

[11] MIT DCI. Utreexo implementation. https://github.com/mit-dci/ utreexo, 2019.
