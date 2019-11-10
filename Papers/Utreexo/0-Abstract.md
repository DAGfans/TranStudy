> Source: https://eprint.iacr.org/2019/611.pdf

# Utreexo: A dynamic hash-based accumulator optimized for the Bitcoin UTXO set

# Utreexo: 针对比特币UTXO集优化的基于哈希的动态累加器

> Thaddeus Dryja tdryja@media.mit.edu
> MIT Digital Currency Initiative

## Abstract 
In the Bitcoin consensus network, all nodes come to agreement on the set of Unspent Transaction Outputs (The “UTXO” set). 
The size of this shared state is a scalability constraint for the network, as the size of the set expands as more users join the system, increasing resource requirements of all nodes.
Decoupling the network’s state size from the storage requirements of individual machines would reduce hardware requirements of validating nodes.
We introduce a hash based accumulator to locally represent the UTXO set, which is logarithmic in the size of the full set.
Nodes attach and propagate inclusion proofs to the inputs of transactions, which along with the accumulator state, give all the information needed to validate a transaction.
While the size of the inclusion proofs results in an increase in network traﬃc, these proofs can be discarded after veriﬁcation, and aggregation methods can reduce their size to a manageable level of overhead.
In our simulations of downloading Bitcoin’s blockchain up to early 2019 with 500MB of RAM allocated for caching, the proofs only add approximately 25% to the amount otherwise downloaded.

## 摘要
在比特币共识网络中，
所有节点都会就未花费交易输出集（“UTXO”集）达成协议
【译注：即每个节点都会在自己本地存储一份一模一样的UTXO集】。
此共享状态【译注：即UTXO集】的大小限制了网络的可伸缩性，
因为集合的大小会随着更多用户加入系统而扩展，从而增加了所有节点的资源需求。
将网络的状态大小与单个计算机的存储需求相解耦可以减少验证节点的硬件需求。
我们引入了一个基于哈希的累加器，
使得节点可以在本地存下能代表UTXO集的数据，
但所占用的存储空间大小相对于整个UTXO集的大小只是对数关系。
服务器节点在传输交易输入时需要带上交易的包含证明，
并与累加器状态一起提供验证交易所需的所有信息。
虽然包含证明的大小会导致网络流量增加，
但是我们可以在验证之后将这些证明丢弃，
并且可以通过聚合的方法将其大小减小到可管理的开销水平。
在我们的模拟测试中，
我们会下载比特币诞生一直到2019年初的比特币区块链，
并分配500MB的RAM用于缓存，
测试结果表明，交易的包含证明仅增加了大约25％的下载量。
