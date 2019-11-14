原文: https://bitslog.files.wordpress.com/2015/09/dagcoin-v41.pdf  

# Periodic re-computing to reduce computation load
# 定期重新计算以减小计算负载

Even using checkpoints, computation load can be high.
When a wallet detects a transaction whose destination address is owned, it will start tracking it to find out how deep it is confirmed.
But computing the confirmation score using every new transaction that arrives is expensive.
To reduce the load, the wallet can re-compute the score after a certain amount of accumulated proof-of-work has been received, creating arbitrary “blocks” of transactions.
Each block is then processed separately to find all the parent transactions “inputs” of the block, and a score is added to each input.
It’s important not to confuse the block inputs with Bitcoin’s UTXOs, DagCoin block inputs are not related with spending and represent block parent hashes (instead of a single patent hash).
The number of inputs will depend on the network latency, but will be generally low and independent of the block size.
For example, for a network with 10 tps, and 1 second of propagation latency, the block input set cardinality should be around 10.
Then the set of inputs is processed.
Figure 3 show a block and how the input set is constructed (not necessarily in the same way) by each wallet software.
Every input has an accumulated score which is propagated to previous nodes.
 
虽然我们使用了检查点，但是计算成本依然很高。
当一个钱包检测到一笔交易的目标地址已经是自己拥有的地址时，它会开始追踪并找出这笔交易的确认深度。
但是计算每一笔新的交易的确认分数的成本是很高的。
为了减小这个计算开销，钱包可以等确认i数达到一个确切的累计工作量证明数量之后再进行计算分数，这样就创造了交易的任意『区块』。
每一个区块都会被分开处理并找到区块『输入』的所有父交易，并且每一个输入都会被添加一个分数。
将区块输入和比特币的 UTXO 的概念区分开来是很重要的，因为 DagCoin 的区块输入与花费和表示父区块的散列（而不是一个单个的父散列）并没有什么关系。
输入的数量取决于网络延迟，但是通常来说都会很低并且独立于区块的大小。
举个例子，对于一个 每秒交易量 为 10 以及传播延迟为 1 秒的网络来说，区块的输入集基数应该是 10 左右。
然后输入集就会被处理。
图表 3 展示了一个区块以及输入集是如何被每一个钱包软件构造出来的（没必要使用同样的方式）。
每一个输入都有一个累计分数，这个累计分数会被传播到之前的节点上。


![Figure 3](https://user-images.githubusercontent.com/22833166/35630516-97a52aee-06dc-11e8-8362-d109d7c0f32e.png)

For instance, and to provide a comparison to Bitcoin, the wallet may consider 10K units of transaction PoW as equivalent 1 “block confirmation” and so pack 10K transactions into a block and re-compute the score every 10K transactions received.
A better approach is to construct a block every N seconds, independent on the number of transactions in it.
Note that if there are no transactions being performed after the monitored one, then the confirmation score does not change.
This is a direct consequence of the nonexistence of a subsidy and Bitcoin will face the same problem if the price does increase constantly.
 
例如，对于比特币来说，钱包或许会认为 10000 个单位的交易工作量证明等于一个『区块确认』，所以它会将 10000 个交易打包进一个区块并对接收到的每 10000 笔交易再次计算分数。
一个更好的方法则是每 N 秒构造一个区块，这个 N 独立于区块中交易的数量。
要注意的一点是，如果被监控的交易之后没有发生任何新的交易那么确认分数就不会改变。
这就是没有经济激励最直接的结果，如果价格持续不断的增长的话，那么比特币也会面临同样的问题。


