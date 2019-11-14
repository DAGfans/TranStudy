原文: https://bitslog.files.wordpress.com/2015/09/dagcoin-v41.pdf  

# Targeting a fixed transactions/rate vs no maximum rate
# 固定交易速率 VS 无最大交易速率

As there are no free-rides for transactions, the transaction/rate is limited by existent deployed computing power and electricity cost.
By time-stamping every transaction, one could dynamically adapt the difficulty of the proof-of-work to achieve more fixed rate.
But if the difficulty of a transactions depends on the difficulty of the parent transactions, then there may be incentives to choose old parent transactions instead of new ones to reduce the PoW required, if the current rate is over the fixed rate.
Just to be sure Moore's law does permit spamming in the future, one could embed a re-targeting rule such that every 18 months the difficulty is doubled.
It seems preferable that the last M transactions (such as M=10K) of a certain transaction vote on an increase or decrease of the difficulty of the following transactions (with small step changes).
Then users could vote more freely on how the network should work without having any immediate benefit to bias voting.
This is a similar problem as the current Bitcoin block-chain increase problem: only miners can vote, because user votes are prone to Sybil attacks.
In DagCoin, every user can vote, as long as it transact.
 
交易是有成本的，交易速率已经被现存计算资源和电力消耗牢牢的限制住了。
通过每一笔交易的时间戳，区块会动态地调整工作量证明的难度来实现更加稳定的速率。
但是如果一个交易的难度取决于它父交易难度的话，当当前的速率高于一个固定的速率时，那么系统就会激励这个交易选择老的父交易，而不是新的，因为这样就不需要工作量证明了。
可以肯定的是，摩尔定律在未来是会允许垃圾交易的，这个定律可以嵌入一个目标规则，比如每 18 个月难度会翻倍。
但是我们更愿意用一个确定的交易最近 M 笔交易（比如M=10K）来决定接下来的交易（只有很小的改变）的难度是增加还是减少。
因为这样用户不会从任何带有偏见的投票中获得直接的收益，那么用户就可以更自由的投票以决定整个网络应该如何运行。
当前比特币的区块链结构也面临同样的问题：只有矿工可以投票，因为用户投票容易被女巫攻击。
在 DagCoin 中，每一个交易过的用户都可以投票。

