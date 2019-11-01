> Source: https://eprint.iacr.org/2019/611.pdf

# Utreexo: A dynamic hash-based accumulator optimized for the Bitcoin UTXO set
# Utreexo: 针对比特币UTXO集优化的基于散列的动态累加器

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
在比特币共识网络中，所有节点都就未花费交易输出集（“ UTXO”集）达成协议。
此共享状态的大小限制了网络的可伸缩性，因为集合的大小会随着更多用户加入系统而扩展，从而增加了所有节点的资源需求。
将网络的状态大小与单个计算机的存储需求分离，将减少验证节点的硬件需求。
我们引入了一个基于散列的累加器来本地表示UTXO集，它的大小与整个集的大小上只是对数关系。
节点附加并传播包含证明到交易输入，并与累加器状态一起提供验证交易所需的所有信息。
虽然包含证明的大小会导致网络流量的增加，但是可以在验证后将这些证明丢弃，并且聚合方法可以将其大小减小到可管理的开销水平。
在我们模拟的下载比特币区块链至2019年初的案例中，分配了500MB的RAM用于缓存，证明仅增加了大约25％的下载量。