> Source: https://eprint.iacr.org/2019/611.pdf
# 1 Introduction
# 1 介绍

As cryptographic currencies such as Bitcoin [1] have seen increased adoption, scalability limitations persist as one of the major drawbacks of the technology.
While Bitcoin’s original author was optimistic that the network would scale well, later research has both tempered that optimism as well as presented novel solutions to reduce the resource requirements for operating the network.

Every node in the network veriﬁes and stores the entire state of the system.
Each user of the system has a wallet, which tracks at least one UTXO,  but generally several.
As the number of users of the system increases, the UTXO set grows, and the resource cost of running a node increases.
This has led to a progressively smaller proportion of users running their own node as more users rely on light clients or on 3rd party nodes to inform them of the state of the network.
Light clients, nodes that do not store the system state and do not validate transactions, can still obtain some assurance about transactions through Simpliﬁed Payment Veriﬁcation (SPV), which leverages Bitcoin’s Proof-of-Work and block commitment scheme to give compact proofs of transaction inclusion into a (not necessarily valid) blockchain.

SPV, while reducing the resource costs of operating a network node, comes with a number of security and privacy weaknesses not present in full nodes.
SPV nodes rely on fully validating nodes to enforce the rules of the system as they cannot do so themselves.
An adversary with suﬃcient hashing power can present transactions which SPV nodes will accept as conﬁrmed, but which will be rejected by fully validating nodes.
While improving the security and privacy of SPV is a promising area of research, this work focuses only on fully validating nodes, and on reducing the resource requirements to run one.

In this paper we present Utreexo, a method for greatly reducing the storage needed to run a fully validating node.
In hardware setups where disk I/O speeds and storage requirements are the bottleneck, this can signiﬁcantly accelerate the validation process, or make validation possible on hardware where it previously has not been.

Utreexo uses a hash-based cryptographic accumulator and introduces a new type of node, the “Compact State Node”, which stores only an accumulator representation of the state.
These nodes require additional inclusion proofs before they are able to verify transactions, and while the CPU time cost of this veriﬁcation is small, the network bandwidth of these proofs is a trade-oﬀ made to achieve the lower state size.
We explore several techniques to limit the size of these proofs.
We observe that the spending patterns of outputs in Bitcoin follow a Pareto distribution where many outputs are short-lived.
We can exploit this locality to keep outputs that are likely to be removed clustered together and use shorter, overlapping proofs when proving their inclusion.
In addition, we are able to cache recent additions, and even anticipate deletions before they occur when catching up to the current network state.
While the proofs add on the order of O(n log n) space overhead to synchronization, in practice with the empirical blockchain data and reasonable caching, about 25% more data needs to be downloaded.

# References

[1] Satoshi Nakamoto. Bitcoin: A peer-to-peer electronic cash system.

http://bitcoin.org/bitcoin.pdf, 2008.