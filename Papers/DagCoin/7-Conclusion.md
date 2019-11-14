原文: https://bitslog.files.wordpress.com/2015/09/dagcoin-v41.pdf  

# Conclusion
# 结论

We’ve presented a new cryptocurrency design based on a DAG structure where there are no fixed blocks and where each transaction carries its own proof of work.
Also we’ve presented two optimizations that allow storing and dynamically updating the DAG-chain consuming low CPU resources.
It must be noted however that the proposed DAG-coin cannot verify new transactions using only a subset of the block-chain, such as Bitcoin’s UTXO set.
However, by storing the most recent transactions in a fast cache, and by using checkpoints where such that older transactions cannot be referenced, the system can be made as fast as Bitcoin, or faster.
  
我们提出了一个基于 DAG 结构的新的加密货币的设计，这个币没有固定的区块，并且它上面的每一笔交易都会为它自己做工作量证明。
同时我们还提出了两个优化方案，这两个优化方案可以允许存储并且仅仅消耗很少的 CPU 资源就可以更新 DAG-链。
但是我们要注意的是，当仅仅使用区块链的一个子集时，例如比特币的 UTXO, 这个币是无法验证新的交易，
然而通过将大部分最近的交易存储在一个快速缓存中，并使用那些更老的不能被引用的交易的检查点，这样就可以让这个系统和比特币一样快了，甚至更快。



