原文: https://bitslog.files.wordpress.com/2015/09/dagcoin-v41.pdf  
作者: [Sergio Demian Lerner](https://twitter.com/sdlerner)

# DagCoin Draft 
## Abstract
## 摘要
 DagCoin is a cryptocurrency design that attempts to be highly decentralized by merging the concepts of transactions and blocks and making each user that transact a miner.
Each transaction carries a proof-or-work and references one or more previous transactions.
The resulting authenticated data structure is a Direct Acyclic Graph (DAG) of transactions where each transaction “confirms” one or more previous transactions.
The confirmation security of a transaction is measured in accumulated amount of proof-of-work referencing the transaction.
In this paper we present the DagCoin design, solve the double-spend problem and show several optimizations to aid for an efficient implementation.
   

 DagCoin 是一个高度去中心化的加密货币的设计，其整合了交易和区块的概念并能让每一个用户成为矿工.
每笔交易都携带一个工作量证明并引用一个或者多个之前的交易.
生成的已认证的数据结构是由交易组成有向无环图（DAG），其中每笔交易 “确认” 一笔或者多笔之前的交易.
交易的确认安全性是以引用该交易的累计工作量证明来衡量的.
在本论文中，我们将介绍 DagCoin 的设计，解决双花问题，并展示几个优化让实现更有效率.

