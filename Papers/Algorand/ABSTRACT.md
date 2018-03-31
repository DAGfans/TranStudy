* **Source:** https://people.csail.mit.edu/nickolai/papers/gilad-algorand-eprint.pdf
* **TranStudy:** https://github.com/DAGfans/TranStudy/edit/master/Papers/Algorand/ABSTRACT.md



# Algorand: Scaling Byzantine Agreements  for Cryptocurrencies

# Algorand: 可扩展的加密货币的拜占庭协议

Yossi Gilad, Rotem Hemo, Silvio Micali, Georgios Vlachos, Nickolai Zeldovich

MIT CSAIL

## ABSTRACT

## 摘要

> Algorand is a new cryptocurrency that confirms transactions with latency on the order of a minute while scaling to many users.
 Algorand ensures that users never have divergent views of confirmed transactions, even if some of the users are malicious and the network is temporarily partitioned.
In contrast, existing cryptocurrencies allow for temporary forks and therefore require a long time, on the order of an hour, to confirm transactions with high confidence.
Algorand uses a new Byzantine Agreement (BA) protocol to reach consensus among users on the next set of transactions. 
To scale the consensus to many users, Algorand uses a novel mechanism based on Verifiable Random Functions that allows users to privately check whether they are selected to participate in the BA to agree on the next set of transactions, and to include a proof of their selection in their network messages. 
In Algorand’s BA protocol, users do not keep any private state except for their private keys, which allows Algorand to replace participants immediately after they send a message. 
This mitigates targeted attacks on chosen participants after their identity is revealed.
We implement Algorand and evaluate its performance on 1,000 EC2 virtual machines, simulating up to 500,000 users.
Experimental results show that Algorand confirms transactions in under a minute, achieves 125×Bitcoin’s throughput, and incurs almost no penalty for scaling to more users.

> Algorand是一种新型的加密货币，在扩展到大量用户的情况下，确认交易的延迟时间仅有一分钟的规模。
Algorand确保用户永远不会对已确认的交易产生分歧，即使某些用户是恶意的并且网络被临时分区。
相比之下，现有的加密货币允许产生临时分叉，因此需要很长时间（大约一小时）才可以较高的可信度确认交易。
Algorand使用新的拜占庭协议（Byzantine Agreement， BA）在下一组交易中达成用户之间的共识。
为了将共识扩展到很多用户，Algorand使用基于可验证随机函数(Verifiable Random Functions)的新机制，允许用户私下检查他们是否被选择参与BA以确认下一组交易，并且在他们的网络消息中包含他们的选择的证明。
在Algorand的BA协议中，用户不保留除私钥以外的任何私有状态，因此允许Algorand在发送消息后立即替换参与者。
在选定的参与者身份曝光后，也可以缓解针对它们的攻击。
我们实现了Algorand并评估了其在1,000个EC2虚拟机上的性能，以模拟多达500,000用户。
实验结果显示，Algorand在不到一分钟的时间内就确认了交易，达到了125倍比特币的吞吐量，并且几乎不会因为扩展到更多用户而受影响。
