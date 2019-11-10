> Source: https://eprint.iacr.org/2019/611.pdf
# 1 Introduction
# 1 介绍

As cryptographic currencies such as Bitcoin [1] have seen increased adoption, scalability limitations persist as one of the major drawbacks of the technology.
While Bitcoin’s original author was optimistic that the network would scale well, later research has both tempered that optimism as well as presented novel solutions to reduce the resource requirements for operating the network.

虽然加密货币（例如比特币[1]）逐渐得到普及，
但可伸缩性限制一直是该技术的主要缺点之一。
比特币的原作者对网络的扩展性持乐观态度，
不过后来的研究一方面给这种乐观态度浇了冷水，
另一方面也提出了新颖的解决方案来减少网络运营所需的资源。

Every node in the network veriﬁes and stores the entire state of the system.
Each user of the system has a wallet, which tracks at least one UTXO,  but generally several.
As the number of users of the system increases, the UTXO set grows, and the resource cost of running a node increases.
This has led to a progressively smaller proportion of users running their own node as more users rely on light clients or on 3rd party nodes to inform them of the state of the network.
Light clients, nodes that do not store the system state and do not validate transactions, can still obtain some assurance about transactions through Simpliﬁed Payment Veriﬁcation (SPV), which leverages Bitcoin’s Proof-of-Work and block commitment scheme to give compact proofs of transaction inclusion into a (not necessarily valid) blockchain.

网络中的每个节点都需要验证并存储整个系统状态。
系统里的的每个用户都有一个钱包，
该钱包可以跟踪至少一个UTXO，但通常要跟踪多个。
随着系统用户数量的增加，UTXO集会变大，并且运行节点的资源成本也会增加。
随着越来越多的用户依赖轻型客户端或第三方节点来同步他们的网络状态，
自己运行服务器节点的用户比例也在逐渐减少。
轻客户端，即不存储系统状态并且不验证交易的节点，
仍然可以通过简化支付验证（SPV）获得一些交易保证，
SPV利用比特币的工作量证明和区块提交机制来提供紧凑的交易证明，
证明交易确实被包含在某条（不一定有效的）区块链中。

SPV, while reducing the resource costs of operating a network node, comes with a number of security and privacy weaknesses not present in full nodes.
SPV nodes rely on fully validating nodes to enforce the rules of the system as they cannot do so themselves.
An adversary with suﬃcient hashing power can present transactions which SPV nodes will accept as conﬁrmed, but which will be rejected by fully validating nodes.
While improving the security and privacy of SPV is a promising area of research, this work focuses only on fully validating nodes, and on reducing the resource requirements to run one.

SPV在降低网络节点的运营资源成本的同时，
也带来了一些在全节点中不存在的安全性和隐私弱点。
SPV节点依靠全节点来执行系统规则，因为它们自己做不了。
这样具有足够的算力的攻击者就可以提出自己构造的恶意交易，
SPV节点会确认接受这些交易，但全节点却会拒绝这些交易。
虽然提高SPV的安全性和隐私性是一个有潜力的研究领域，
但本文的工作仅关注全节点，以及如何降低一个全节点所需的资源。

In this paper we present Utreexo, a method for greatly reducing the storage needed to run a fully validating node.
In hardware setups where disk I/O speeds and storage requirements are the bottleneck, this can signiﬁcantly accelerate the validation process, or make validation possible on hardware where it previously has not been.

在本文中，我们介绍了Utreexo，
这是一种极大减少全节点所需的存储空间的方法。
在磁盘I/O速度和存储要求成为瓶颈的硬件配置中，
这可以显著加快验证过程，
或者使以前因为硬件限制而不可行的验证变为可能。

Utreexo uses a hash-based cryptographic accumulator and introduces a new type of node, the “Compact State Node”, which stores only an accumulator representation of the state.
These nodes require additional inclusion proofs before they are able to verify transactions, and while the CPU time cost of this veriﬁcation is small, the network bandwidth of these proofs is a trade-oﬀ made to achieve the lower state size.
We explore several techniques to limit the size of these proofs.
We observe that the spending patterns of outputs in Bitcoin follow a Pareto distribution where many outputs are short-lived.
We can exploit this locality to keep outputs that are likely to be removed clustered together and use shorter, overlapping proofs when proving their inclusion.
In addition, we are able to cache recent additions, and even anticipate deletions before they occur when catching up to the current network state.
While the proofs add on the order of O(n log n) space overhead to synchronization, in practice with the empirical blockchain data and reasonable caching, about 25% more data needs to be downloaded.

Utreexo使用基于哈希的密码学累加器，
并引入了一种名为“紧凑状态节点”的新型节点，
该节点用累加器表示整个系统的状态，
并仅将累加器保存在存储中。
这些节点在能够验证交易之前需要额外的包含证明，
这些验证的CPU时间开销很小，
不过会占用网络带宽，
但这也是为了减小系统状态大小而必须付出的成本。
我们会探索几种技术来限制这些证明的大小。
我们发现比特币产出的花费模式遵循Pareto分布，
其中许多输出很快就会被花费。【译注：80/20法则】
我们可以利用这种局部性将有可能成片删除掉的UTXO（即生存时间短的UTXO）聚集在一起，
并在提供它们的包含证明时使用较短的重叠证明。
【译注：重叠是指这个证明可以复用, 可以同时证明多笔交易，详见第5节。】
此外，我们可以缓存最近添加的内容，
甚至可以在同步当前网络状态前预测会被删除的内容。
虽然证明增加了O(n log n)数量级的空间开销来进行同步，
但实际上如果能利用区块链的先验数据并合理地分配缓存，
那么下载流量大约只会增加25％。

# References

[1] Satoshi Nakamoto. Bitcoin: A peer-to-peer electronic cash system.

http://bitcoin.org/bitcoin.pdf, 2008.
