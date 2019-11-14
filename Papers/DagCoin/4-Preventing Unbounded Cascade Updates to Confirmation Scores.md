原文: https://bitslog.files.wordpress.com/2015/09/dagcoin-v41.pdf  

# Preventing Unbounded Cascade Updates to Confirmation Scores  
# 防止无限级联更新确认分数

Suppose that for each transaction we save an integer score that we update for each new  transaction that references it directly or recursively.
It is evident that the proposed data structure requires updating almost all previous confirmation scores each time a transaction is added.
To reduce the workload, we use pointers and checkpoints.
At a certain frequency the software chooses a transaction that references a high number of parent nodes.
 
假设对于每个交易，我们保存一个整数分数，每当有新的交易直接或递归地引用它时, 我们需要更新它。
显然, 每当一笔交易加入的时候, 提交的数据结构需要更新几乎所有之前的确认分数。
为了减少工作量，我们使用指针和检查点。
假设在某个频率上，软件选择一笔引用大量父节点的交易。


![Figure 2](https://user-images.githubusercontent.com/22833166/35630410-4e1f29ec-06dc-11e8-91a3-98c4354204f3.png)

> Figure 2 show how a checkpoint is found.

> 图 2 显示了如何找到检查点。

Of course, not every past transaction could be reachable, as users may decide to never reference certain published transaction.
However, the parent selection, with average out-degree 2, and low network latency, can guarantee that there will be frequent checkpoints referencing almost all previous transactions.
 
当然，并不是每一个过去的交易都是可以到达的，因为用户可能决定从不引用某个已发布的交易。
 然而，在平均出度为 2 和低网络延迟的情况下, 父交易的选择可以保证将会有频繁的检查点引用几乎所有以前的交易。


After a checkpoint is found, the software updates all nodes reachable from the checkpoint with a forward pointer to this checkpoint.
A checkpoint has its own score counter, initially set to zero.
When the update algorithm reaches a checkpoint node, it increments the counter and stops propagating backward.
The score of a transaction is computed as the last stored score in the transaction plus the score of the pointed checkpoint.
Checkpoints are considered as nodes on the DAG, so the same check-
point finding algorithm can make checkpoints that refer to other forward checkpoints.
After several continuous checkpoints, a checkpoints of a higher level is created referencing previous checkpoints, forming a skip-list.
Using the skip-list it is possible to compute the score of a transaction in O(logN)
where N is the number of transactions after it.
Also, after a certain score has been reached, the wallet may decide not to update it anymore and consider it immutable, as in Bitcoin checkpoints.
 
找到检查点之后，软件会从指向的这个检查点的检查点开始更新所有可到达的节点。
每一个检查点有它自己的分数计数器，这个计数器初始化的时候被设置为 0。
当我们的更新算法到达一个检查点节点时，它会增加这个计数器并停止向后传播。
一笔交易的得分是由交易中最后存储的分数和被指向检查点的分数相加得到的。
这些检查点在 DAG 中被当作节点，所以同样的检查点查找算法也可以找到和之前的检查点有联系的检查点。
在一些连续的检查点之后，一个引用了之前所有检查点的更高等级的检查点就会被创建出来，这样就形成一个忽略列表。
使用这个忽略列表可以在 O(logN) 的复杂度下计算出一笔交易的分数，这里的 N 是这个检查点之后交易的数量。
同时，在达到一个确切的分数之后，钱包会决定不再更新它并把它当做不可变的，就像比特币中的检查点一样。

