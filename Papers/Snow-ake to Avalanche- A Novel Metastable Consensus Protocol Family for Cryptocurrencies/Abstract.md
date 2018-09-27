## Abstract
## 摘要

This paper introduces a new family of leaderless Byzantine fault tolerance protocols, built on a metastable mechanism. 
这篇文章介绍了一个新的无领导拜占庭容错共识协议系列，基于亚稳态机制。
These protocols provide a strong probabilistic safety guarantee in the presence of Byzantine adversaries, while their concurrent nature enables them to achieve
high throughput and scalability. 
这些协议在拜占庭敌人存在的情况下提供了安全保障,而他们的并发性使他们能够实现高吞吐量和可扩展性。
Unlike blockchains that rely on proof-of-work, they are quiescent and green. 
不像依赖工作量证明的区块链,它们是静止的和绿色的。
Surprisingly, unlike traditional consensus protocolswhich require $O(n^2)$ communication, their communication complexity ranges from $O(knlogn)$ to $O(kn)$ for some security parameter $k\ll n$.
令人惊讶的是，与需要$O(n^2)$复杂度通信的传统共识协议不同，它们的通信复杂度范围从$O(knlogn)$到$O(kn)$，对于某些安全性参数$k\ll n$。

The paper describes the protocol family, instantiates it in three separate protocols, analyzes their guarantees, and describes how they can be used to construct the core of an internet-scale electronic payment system. 
文章描述了协议族，使它在三个独立的协议中实例化，分析他们的保章机制，并且描述如何使用它们来构建互联网规模的电子支付系统的核心。
The system is evaluated in a large scale deployment. 
系统在大规模调度中进行评估。
Experiments demonstrate that the system can achieve high throughput (1300 tps), provide low conﬁrmation latency (4 sec), and scale well compared to existing systems that deliver similar functionality. 
实验表明与提供类似功能的现有系统相比，该系统能实现高吞吐量(1300 tps)，提供低证明延迟(4秒)和扩展良好。
For our implementation and setup, the bottleneck of the system is in transaction veriﬁcation.
对于我们的实施和设置，系统的瓶颈在于交易验证。
