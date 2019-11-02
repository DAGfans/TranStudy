> Source: https://eprint.iacr.org/2019/611.pdf
# 1 Introduction
# 1 介绍

As cryptographic currencies such as Bitcoin [1] have seen increased adoption, scalability limitations persist as one of the major drawbacks of the technology.
While Bitcoin’s original author was optimistic that the network would scale well, later research has both tempered that optimism as well as presented novel solutions to reduce the resource requirements for operating the network.

随着加密货币（例如比特币[1]）的采用率不断提高，可伸缩性限制一直是该技术的主要缺点之一。
尽管比特币的原始作者对网络能够很好地扩展持乐观态度，但后来的研究既减轻了人们的乐观情绪，也提出了新颖的解决方案来减少网络运营所需的资源。

Every node in the network veriﬁes and stores the entire state of the system.
Each user of the system has a wallet, which tracks at least one UTXO,  but generally several.
As the number of users of the system increases, the UTXO set grows, and the resource cost of running a node increases.
This has led to a progressively smaller proportion of users running their own node as more users rely on light clients or on 3rd party nodes to inform them of the state of the network.
Light clients, nodes that do not store the system state and do not validate transactions, can still obtain some assurance about transactions through Simpliﬁed Payment Veriﬁcation (SPV), which leverages Bitcoin’s Proof-of-Work and block commitment scheme to give compact proofs of transaction inclusion into a (not necessarily valid) blockchain.

网络中的每个节点都将验证并存储系统的整个状态。
系统的每个用户都有一个钱包，该钱包可以跟踪至少一个UTXO，但通常是几个。
随着系统用户数量的增加，UTXO集会增加，并且运行节点的资源成本也会增加。
随着越来越多的用户依赖轻型客户端或第三方节点来通知他们网络状态，这导致运行自己的节点的用户比例逐渐减少。
轻客户端，即不存储系统状态并且不验证交易的节点，仍然可以通过简化支付验证（SPV）获得一些交易保证，该交易利用比特币的工作量证明和区块承诺方案来提供紧凑的交易证明 包含在（不一定有效的）区块链中。(译注：承诺方案是先提交承诺但不公开等到一定条件后再公开的方案，可以用于交互式的加密协议，这里的区块承诺方案具体还不知道是什么，后面应该会有说明)

SPV, while reducing the resource costs of operating a network node, comes with a number of security and privacy weaknesses not present in full nodes.
SPV nodes rely on fully validating nodes to enforce the rules of the system as they cannot do so themselves.
An adversary with suﬃcient hashing power can present transactions which SPV nodes will accept as conﬁrmed, but which will be rejected by fully validating nodes.
While improving the security and privacy of SPV is a promising area of research, this work focuses only on fully validating nodes, and on reducing the resource requirements to run one.

SPV在降低操作网络节点的资源成本的同时，还带来了一些在完整节点中不存在的安全性和隐私弱点。
SPV节点依靠全节点来执行系统规则，因为它们自己做不了。
具有足够的算力的对手可以提出交易，SPV节点将确认接受这些交易，但这些交易将被全节点拒绝。
虽然提高SPV的安全性和隐私性是一个有前途的研究领域，但本项工作仅侧重于全节点，以及降低运行一个节点的资源要求。

In this paper we present Utreexo, a method for greatly reducing the storage needed to run a fully validating node.
In hardware setups where disk I/O speeds and storage requirements are the bottleneck, this can signiﬁcantly accelerate the validation process, or make validation possible on hardware where it previously has not been.

在本文中，我们介绍了Utreexo，这是一种极大减少运行全节点所需的存储量的方法。
在磁盘I/O速度和存储要求成为瓶颈的硬件设置中，这可以显着加快验证过程，或者使以前不可能行过验证的硬件成为可能

Utreexo uses a hash-based cryptographic accumulator and introduces a new type of node, the “Compact State Node”, which stores only an accumulator representation of the state.
These nodes require additional inclusion proofs before they are able to verify transactions, and while the CPU time cost of this veriﬁcation is small, the network bandwidth of these proofs is a trade-oﬀ made to achieve the lower state size.
We explore several techniques to limit the size of these proofs.
We observe that the spending patterns of outputs in Bitcoin follow a Pareto distribution where many outputs are short-lived.
We can exploit this locality to keep outputs that are likely to be removed clustered together and use shorter, overlapping proofs when proving their inclusion.
In addition, we are able to cache recent additions, and even anticipate deletions before they occur when catching up to the current network state.
While the proofs add on the order of O(n log n) space overhead to synchronization, in practice with the empirical blockchain data and reasonable caching, about 25% more data needs to be downloaded.

Utreexo 使用基于散列的密码累加器，并引入了一种新型节点“紧凑状态节点”，该节点仅存储状态的累加器表示形式。
这些节点在能够验证交易之前需要额外的包含证明，并且尽管此验证的CPU时间开销很小，但是这些证明占用的网络带宽是为了达到较低状态大小而付出的成本。
我们探索几种技术来限制这些证明的大小。
我们观察到，比特币中产出的花费模式遵循Pareto分布，其中许多输出是短暂的。(译注：80/20法则)
我们可以利用这种局域性来将可能要删除的输出聚集在一起，并在证明它们包含时使用较短的重叠证明。(译注：重叠这里还不是很明确什么意思，应该是指这个证明可以复用, 可以同时证明多个数据)
此外，我们能够缓存最近添加的内容，甚至可以在赶上当前网络状态时在预测删除的内容。
虽然证明增加了O（n log n）空间开销的数量级来进行同步，但实际上是在经验性的区块链数据和合理的缓存的情况下，大约只需要多下载25％的数据。

# References

[1] Satoshi Nakamoto. Bitcoin: A peer-to-peer electronic cash system.

http://bitcoin.org/bitcoin.pdf, 2008.