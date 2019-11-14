原文: https://bitslog.files.wordpress.com/2015/09/dagcoin-v41.pdf  

# Preventing too many transactions merging too many transactions  
# 防止太多的交易合并太多的交易

The core idea proposed is that each transaction commits to an authenticated forest of previous unreferenced transactions.
To do so, it includes the value C(N), where C(i)=Commit(C(i-1) || T(i)), where T(i) is the hash of a transaction parent and C(0) is the empty string.
These are simple recursive commitments so that C(N) allows the payer to reveal any number of parent hashes between 1 to N.
The important decision is how many parents the commitment should reveal.
Using the transaction as a header, the payer tries to find a proof of work with certain base difficulty (more on this base difficulty later).
If the obtained a proof-of-work whose difficulty is 2^k times harder than the base difficulty, it will reveal and reference the first (k+1) nodes of the list.
Half of the times a transaction will have a single parent, so only the first node T(N) will be revealed, by providing the complementary hash chain head (C(N-1)).
One fourth of the times, two transactions will be referenced, by providing the hashes T(N), T(N-1) and C(N-2).
 
该提案的核心思想是，每个交易都提交一个以前未被引用的已认证的交易集。
为此，它包括值 C(N)，其中 C(i)=Commit(C(i-1) || T(i))，其中 T(i)是父交易的散列，C(0)是空字符串。
这些都是简单的递归提交，因此 C(N)允许支付方在 1 到 N 之间公开任意数量的父散列。
但重要的是要决定提交应公开多少父散列。
要使用这个交易作为头部，支付方需要试图找到满足某些基本难度的工作证明（以后这个基本难度会更大）。
如果获得了难度是基本难度 2^k 倍的工作证明，它将公开和引用列表的第 (k+1) 个节点。
通过提供剩余的散列链头 (C(N-1))，交易在一半的机会下的将拥有单个父节点，所以只公开第一个节点 T(N)。
四分之一的机会里，通过提供哈希值 T(N)，T(N-1) 和 C(N-2), 两个交易会被引用。


This system provides a logarithm distribution in the amount of parents, with an average of 2.
Also this method cannot be gamed, since referring more parents has a PoW cost.
 
这个系统提供了呈一个对数分布的父交易的数量，平均为 2.
这种方法不能被操纵，因为引用更多的父交易需要 PoW 成本。


There should be an incentive to include as many references as possible in the authenticated branch. 
This can be achieved by several methods:  
需要有一个激励, 让已认证的分支中包含尽可能多的引用。
这可以通过几种方法来实现：

1. Invalidating a transaction that has less references than what the PoW requires.
2. Incrementing the score of a transaction that has more revealed references.
For instance, a transaction having K revealed references could add a fractional score of (K-1)/K to the transaction score.
    
>
1. 如果少于 PoW 要求的引用, 则交易是无效的。
2. 增加有更多已公开引用的交易的分数。
 例如，有 K 个已公开引用的交易可以增加 (K-1)/K 倍分数到交易分数上。
