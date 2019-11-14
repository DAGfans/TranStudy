原文: https://bitslog.files.wordpress.com/2015/09/dagcoin-v41.pdf  

# Safely accepting Double-spends in the DAG-chain
# 安全地接受 DAG 链中的双花

In Bitcoin, a transaction in a valid block-chain can never be a double-spend, as double-spending violates a protocol rule.
DagCoin allows two conflicting transactions to be included in the DAG-chain as long as the second does not references the first (over one or more hops).
We assign each transaction a confirmation score.
If two conflicting transactions appear, as more transactions are added to the DAG-chain, the number of confirmations of one of the two will increase, but the other will not.
Each transaction adds one unit of confirmation.
The score of a node without children is zero.
The score of a referenced transaction is the sum of all transactions that recursively reference it (including double-spends).
Whenever a transaction is added, it modifies the scores of all transactions recursively referenced by it.
Whenever a transaction references a list of previous transactions, if there are two conflicting transactions, then the one with highest score prevails.
If both have the same score, then the order of referencing establishes preferences over the conflicting transactions, such that the first transaction gets its score increased but any following double-spend will not.
 
在比特币中，有效区块链中是不可能存在双花交易的，因为双花违反协议规则.
DagCoin 允许两个冲突交易包含在 DAG 链中，只要第二个交易不引用第一个（直接或者间接）.
我们为每笔交易分配一个确认分数.
如果出现两个相冲突的交易，随着更多的交易被添加到 DAG 链中，两个中的一个的确认数量将增加，而另一个不会.
每笔交易增加一个确认单位.
没有孩子的节点的分数是零.
引用交易的分数是递归引用的所有交易的总和（包括双花）.
无论何时添加交易，都会修改由其递归引用的所有交易的分数.
每当一笔交易引用一笔以前的交易时，如果有两笔冲突的交易，则以最高分为准.
如果两者都具有相同的分数，则用引用的顺序来建立冲突交易的优先级, 使得第一笔交易分数增加但是任何后续的双花则不会.

![Figure 1](https://user-images.githubusercontent.com/22833166/35629720-4ff83404-06da-11e8-91bb-7fe5f72164db.png)

> Figure 1 shows the DAG before a join transaction arrives and afterward.
Transactions 2 and 3 (in orange) are conflicting.
The confirmation score is in brackets.
We can see that even both transaction 2 and transaction 3 have a non-zero confirmation score, only one of them will increase over time.Honest nodes will never extend a transaction which is already referenced, so an attacker that wants to replace transaction 2 by transaction 10 must invest in proof-of-work at least the difference between the confirmation scores.
This establishes a very precise bound on the double-spend security.
 
> 图 1 显示了新加入的交易到达之前和之后的 DAG.
交易 2 和 3（橙色）有冲突.
确认分数在括号内.
我们可以看到，即使交易 2 和交易 3 都有一个非零的确认分数，但其中只有一个会随着时间的推移而增加.
诚实的节点永远不会扩展已经被引用的交易，所以攻击者想要用交易 10 替换交易 2, 则必须至少要投入确认分数差这么多工作证明.
这就确立了抵抗双花攻击的非常精确的安全边界.
