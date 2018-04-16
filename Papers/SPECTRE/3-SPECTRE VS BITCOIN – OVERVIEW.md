> **Source：** [https://eprint.iacr.org/2016/1159.pdf](https://eprint.iacr.org/2016/1159.pdf)  
> **TranStudy：** [https://github.com/DAGfans/TranStudy/new/master/Papers/SPECTRE](https://github.com/DAGfans/TranStudy/new/master/Papers/SPECTRE)

# 3. SPECTRE VS BITCOIN – OVERVIEW
# 3. SPECTRE VS BITCOIN – 概述


SPECTRE adopts many of Bitcoin’s solution features. 
In particular, miners create blocks, which are batches of transactions. 
A valid block must contain a solution to the PoW puzzle (Bitcoin for example, uses PoW that is based on partial SHA256 collisions). 
The block creation rate, denoted λ, is kept constant by the protocol by occasional readjustments of the PoW difﬁculty; 
we elaborate on this mechanism in SPECTRE in Appendix D. 
The size of a block is limited by some B KB.

SPECTRE采用了比特币的许多解决方案的功能。 
特别是，矿工创建块，即成批的交易。 
一个有效的区块必须包含PoW难题的解（以比特币为例，使用基于部分SHA256冲突的PoW）。
(**译注：** 这里的部分冲突指的生成的散列值不要求每一位都一样，只要部分一样就可以了)
出块率，记为λ，通过协议定期重新调整PoW难度而保持不变; 
我们在附录D中详细说明了SPECTRE的这种机制。
块的大小限制为某个值B KB。

Bitcoin’s throughput can be increased by increasing either the block size limit (which in turn increases D) or\and the block creation rate λ. 
Alas, it is well established that the security threshold of Nakamoto Consensus deteriorates as D · λ increases:

比特币的吞吐量可以通过增加块大小限制（反过来会增加D）或\和出块率λ来增大。 
但是，经过充分地论证，中本聪共识的安全阈值会随着D·λ的增加而恶化：
(**译注：** 这里的D其实指的是区块传播到全网的时间，所以增加区块大小会增加D，不知道为什么论文没有提前说明就直接用了)

### Theorem 2. [Bitcoin is not scalable] The security threshold of the Bitcoin protocol goes to zero as D · λ increases.

### 定理 2. [比特币是不可扩展的] 比特币协议的安全阈值随着D·λ增加而变为零。

The proof of this theorem appears in various forms in previous works, see [18], [15], [7]. 
To maintain a high security threshold, Bitcoin suppresses its throughput by keeping λ low – 1/600 blocks per second. 
This large safety margin is needed because λ (and B ) are decided once and for all at the inception of the protocol. Consequently, even when the network is healthy and D is low, Bitcoin suffers from a low throughput – 3 to 7 transactions per second, and slow conﬁrmation times – tens of minutes. 
In contrast, SPECTRE’s throughput can be increased without deteriorating the security threshold:

这个定理的证明在之前的工作中以各种形式出现，参见[18]，[15]，[7]。 
为了保持较高的安全阈值，比特币通过保持低λ - 每秒1/600块来抑制其吞吐量。 
这个较大的安全系数是必要的，因为λ（和B）是在协议在开始时决定后就不能更改了。 
因此，即使网络健康且D较低，比特币的吞吐量也很低 - 每秒3到7次交易，确认时间也很慢 - 几十分钟。 
相反，SPECTRE的吞吐量可以在不降低安全阈值的情况下增加：

### Theorem 3. [SPECTRE is scalable] For any D · λ, SPECTRE’s security threshold is 50%.

### 定理 3. [SPECTRE 是可扩展的] 对于任何D·λ，SPECTRE的安全阈值为50％。

Therefore, in the context of the Distributed Algorithms literature, SPECTRE falls into the partial synchronous setup, as it remains secure for any value of D. Theorem 3 is proven rigorously in Appendix E.

因此，根据分布式算法文献的分类，SPECTRE的设定属于部分同步，因为对任何D值都是安全的。定理3在附录E中得到了严格证明。
(**译注：** 部分同步是指D值虽然存在网络延迟的上限，但是我们事先不知道。比如说网络实际上要1分钟才能传播到大部分节点，但因为我网络状况比较好，我就估算只要半分钟，这样会造成只有网络状况好的部分节点才能和我保持同步，其他的节点就同步不了了，所以成为部分同步。)

Of course, λ cannot be increased indeﬁnitely or otherwise the network will be ﬂooded with messages (blocks) and become congested. 
Theorem 3 “lives” in the theoretical framework (speciﬁed in Section 2), which does not model the limits on nodes’ bandwidth and network capacity. 
Practically, these barriers allow for a throughput of thousands of transactions per second, by setting λ = 10 and b = 100, for instance. 
For further discussion refer to Appendices B and D.

当然，λ不能无限增加，否则网络将被消息（区块）淹没并变得拥塞。 
定理3建立理论框架之上（在第2节中已具体说明），它没有模拟节点带宽和网络容量的限制。 
实际上，在这些障碍下，协议还是可以允许每秒处理数千次交易，例如，通过设置λ= 10和b = 100。 
有关进一步的讨论，请参阅附录B和D.

Asymptotically, SPECTRE’s conﬁrmation times are in $\mathcal{O}(\frac{ln(1/ϵ)}{λ(1-2α)}+\frac{D}{1-2α})$ . 
In practice, this allows for conﬁrmation times of mere seconds, under normal network conditions. 
When running RobustTxO, each node in SPECTRE uses its own upper bound on the recent D in the network. 
This bound affects only its own operation—underestimating D will result in premature acceptance of transactions, and overestimating it by far will delay acceptance unnecessarily (by a time linear in the difference). 
Importantly, in case of network hiccups and long network delays, the node can switch in his local client to a more conservative bound on D without coordinating this with other nodes.

渐近地，SPECTRE的确认时间在$\mathcal{O}(\frac{ln(1/ϵ)}{λ(1-2α)}+\frac{D}{1-2α})$。
(**译注：** $\mathcal{O}$  代表趋近，上面的公式可以表示为确认时间趋近于$\frac{ln(1/ϵ)}{λ(1-2α)}+\frac{D}{1-2α}$，目前还不理解这个公式是怎么来的)
实际上，这允许在正常网络条件下仅仅几秒钟的确认时间。
运行RobustTxO时，SPECTRE中的每个节点都会在网络中最近的D上使用自己的上限。
这种限制只会影响其自身的操作 - 低估D会导致交易过早被接受，并且过高估计它将会导致不必要地延迟接受（以时间线性差异）。
(**译注：** 这里的意思就是每个节点根据自身网络状况，预估参数D，因此每个节点的D可以不一样)
重要的是，在网络不稳定和网络延迟较长的情况下，节点可以将其本地客户端的D上限切换得更保守，而无需与其他节点进行协调。
