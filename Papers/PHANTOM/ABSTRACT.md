> **Source：** [https://eprint.iacr.org/2018/104.pdf](https://eprint.iacr.org/2018/104.pdf)  
> **TranStudy：** [https://github.com/DAGfans/TranStudy/edit/master/Papers/PHANTOM/Abstract.md](https://github.com/DAGfans/TranStudy/edit/master/Papers/PHANTOM/Abstract.md)

# PHANTOM: A Scalable BlockDAG protocol

# PHANTOM: 可扩展的 BlockDAG 协议

### Yonatan Sompolinsky and Aviv Zohar

### School of Engineering and Computer Science,

### 计算机科学与工程学院

### The Hebrew University of Jerusalem, Israel

### 耶路撒冷希伯来大学，以色列

### {yoni sompo,avivz}@cs.huji.ac.il

## Abstract

> In 2008 Satoshi Nakamoto invented the basis for blockchain based distributed ledgers. The core concept of this system is an open and anonymous network of nodes, or miners, which together maintain a public ledger of transactions. The ledger takes the form of a chain of blocks, the blockchain, where each block is a batch of new transactions collected from users. One primary problem with Satoshi’s blockchain is its highly limited scalability. The security of Satoshi’s longest chain rule, more generally known as the Bitcoin protocol, requires that all honest nodes be aware of each other’s blocks very soon after the block’s creation. To this end, the throughput is artificially suppressed so that each block fully propagates before the next one is created, and that very few “orphan blocks” that fork the chain be created spontaneously. 
> In this paper we present PHANTOM, a protocol for transaction confirmation that is secure under any throughput that the network can support. PHANTOM thus does not suffer from the security-scalability tradeoff which Satoshi’s protocol suffers from. PHANTOM utilizes a Directed Acyclic Graph of blocks, a blockDAG, a generalization of Satoshi’s chain which better supports faster block generation and larger blocks that take longer to propagate. PHANTOM uses a greedy algorithm on the blockDAG to distinguish between blocks mined properly by honest nodes and those mined by non-cooperating nodes who chose to deviate from the mining protocol. Using this distinction, PHANTOM provides a robust full order on the blockDAG in a way that is eventually agreed upon by all honest nodes.

## 摘要

> 2008年，中本聪发明了基于区块链的分布式账本的基础。 这个系统的核心概念是一个开放和匿名的节点或矿工网络， 它们共同维护着交易的公共账本。 账本采取区块的链式形式，即区块链，每个区块是从用户收集的一批新交易。 中本聪式区块链的一个主要问题是其可扩展性非常有限。 中本聪的最长链规则(通常称为比特币协议)的安全性，要求所有诚实的节点都会在块创建后很快了解到对方的块。 为此，吞吐量被人为地抑制，使得每个块在下一个块被创建之前可以完全传播，并且很少的分叉产生的“孤块”可以自发创建。 
> 在本文中，我们介绍PHANTOM，用于交易确认的协议，在网络可支持的任何吞吐量下都是安全的。 因此，PHANTOM没有中本聪协议所面临的要在安全和可扩展性之间权衡的问题。 PHANTOM利用有向无环图区块，又称blockDAG，一个更适合更快速的块增长和传播时间更长的更大区块的广义上的中本聪式区块链。 PHANTOM在blockDAG上使用贪婪算法来区分诚实节点正确挖出的区块和选择偏离DAG采矿协议的非协作节点挖出的区块。 利用这个区别，PHANTOM以最终由所有诚实节点同意的方式在blockDAG上提供全序。
所有诚实的节点都会在块创建后很快意识到对方的块。 为此，吞吐量被人为地抑制，使得每个块在下一个块被创建之前可以完全传播，并且没有任何分叉产生的“孤块”可以自发创建。 在本文中，我们介绍PHANTOM，用于交易确认的协议，在网络可支持的任何吞吐量下都是安全的。 因此，PHANTOM没有中本聪协议所面临的要在安全和可扩展性之间权衡的问题。 PHANTOM利用有向无环图区块，又称blockDAG，一个更适合快速或大区块配置下的广义上的中本聪式区块链。 PHANTOM在blockDAG上使用贪婪算法来区分诚实节点正确挖出的区块和偏离DAG采矿协议的非协作节点挖出的区块。 利用这个区别，PHANTOM以最终由所有诚实节点同意的方式在blockDAG上提供强大的全序。
