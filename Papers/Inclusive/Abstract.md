> Source: https://www.cs.huji.ac.il/~yoni_sompo/pubs/15/inclusive_full.pdf

# Inclusive Block Chain Protocols
# Inclusive 区块链协议 

Yoad Lewenberg<sup>1</sup> , Yonatan Sompolinsky<sup>1</sup>, and Aviv Zohar<sup>1,2</sup>

<sup>1</sup> The School of Engineering and Computer Science,

The Hebrew University of Jerusalem, Israel 

<sup>2</sup> Microsoft Research, Herzliya, Israel 

{yoadlew,yoni sompo,avivz}@cs.huji.ac.il

## Abstract. 
Distributed cryptographic protocols such as Bitcoin and Ethereum use a data structure known as the block chain to synchronize a global log of events between nodes in their network. 
Blocks, which are batches of updates to the log, reference the parent they are extending, and thus form the structure of a chain. 
Previous research has shown that the mechanics of the block chain and block propagation are constrained: if blocks are created at a high rate compared to their propagation time in the network, many conﬂicting blocks are created and performance suffers greatly. 
As a result of the low block creation rate required to keep the system within safe parameters, transactions take long to securely conﬁrm, and their throughput is greatly limited.

## 摘要 
诸如比特币和以太坊之类的分布式密码协议使用称为区块链的数据结构来同步其网络中节点之间事件的全局日志。 
区块是对日志的批量更新，区块通过引用父节点将链不断延伸，因此形成了链式结构。 
先前的研究表明，区块链和区块传播的机制受到限制：如果与网络中的传播时间相比，以较高的速率创建区块，则会创建许多冲突的区块，并且性能会大大降低。 
由于将系统保持在安全参数范围内所需的块创建率较低，因此交易需要很长时间才能安全确认，并且其吞吐量受到极大限制。

We propose an alternative structure to the chain that allows for operation at much higher rates. 
Our structure consists of a directed acyclic graph of blocks (the block DAG). 
The DAG structure is created by allowing blocks to reference multiple predecessors, and allows for more “forgiving” transaction acceptance rules that incorporate transactions even from seemingly conﬂicting blocks. 
Thus, larger blocks that take longer to propagate can be tolerated by the system, and transaction volumes can be increased.

我们提出了一种链的替代结构，该结构允许以更高的交易速度运行。 
我们的结构由区块的有向无环图（Block DAG）组成。 
通过允许区块引用多个前置区块来创建DAG结构，并允许更“宽容”的交易接受规则，这些规则甚至能从看似冲突的区块中将交易融合进来。 
因此，该系统可以接受需要较长时间传播的较大块，并且可以增加交易吞吐量。

Another deﬁciency of block chain protocols is that they favor more connected nodes that spread their blocks faster—fewer of their blocks conﬂict. 
We show that with our system the advantage of such highly connected miners is greatly reduced. 
On the negative side, attackers that attempt to maliciously reverse transactions can try to use the forgiving nature of the DAG structure to lower the costs of their attacks. 
We provide a security analysis of the protocol and show that such attempts can be easily countered.

区块链协议的另一个不足之处是，网络状况较好的节点有更大的优势，这些节点可以更快地传播其区块，从而减少块冲突。 
我们证明，使用我们的系统，这种网络状况较好的矿工的优势大大降低。 
不利的一面是，试图恶意逆转交易的攻击者可以尝试利用DAG结构的宽容性质来降低攻击成本。 
我们提供了对该协议的安全性分析，并表明可以很容易地阻止此类尝试。