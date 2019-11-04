> Source: https://eprint.iacr.org/2019/611.pdf
# 2 Related Work
# 2 相关工作

The limitations of having every client store the UTXO set have been clear since Bitcoin was ﬁrst introduced; in fact this was the ﬁrst comment ever recorded about Bitcoin [2].
Earlier work has focused on UTXO commitments [3], where some representation of the UTXO set would be included in the blockchain itself.
With a consensus rule enforcing the validity of this UTXO commitment, users could get some assurance, similar to SPV security, about the representation of the UTXO set without downloading or validating it themselves.
In contrast to UTXO commitments, Utreexo state is not transmitted to clients, but instead computed by them.

自从首次引入比特币以来，让每个客户端都存储UTXO集的局限性就很明显了。实际上，这是有史以来关于比特币的第一个评论[2]。
早期的工作专注在UTXO提交[3]上，其中UTXO集以某种表示形式被包含在区块链本身中。
通过一种强制保证此UTXO提交有效性的共识规则，用户可以像SPV安全性一样，获得有关UTXO集表示形式的某种保证，而无需自己下载或验证UTXO集。
与UTXO提交相反，Utreexo状态不是传输给客户端的，而是由客户端自己计算得出的。

Techniques for speeding up or skipping parts of Initial Block Download have been proposed, the most promising of which is Mimblewimble [4].
While Mimblewimble maintains the same cryptographic security as downloading and verifying the transaction history, it is a substantial change in transaction format and is difficult to implement in a backwards-compatible way to the existing Bitcoin network.
Systems which do implement Mimblewimble beneﬁt from a decrease in data needed to perform initial synchronization, but once synchronized still maintain a full UTXO set.
It’s possible that Utreexo and Mimblewimble techniques could be implemented simultaneously, which would allow for both reduced state size and efficient initial synchronization.

已经有人提出了加快或部分跳过“初始区块下载”的技术
【译注：*初始区块下载*又叫初始同步。英文原文是Initial Block Download（IBD），
是指一个全节点初次上线，在其有能力校验一个未确认的交易和最近新挖到的区块前，
必须下载并校验从创世区块起的整个交易历史记录的初始化过程】，
其中最具代表性的是Mimblewimble[4]。
尽管Mimblewimble在没有下载和验证整个交易历史记录的情况下依然可以保持同等的密码学安全性，
但其交易格式产生了重大变化，并且难以以向后兼容的方式应用到现有的比特币网络中。
那些确实实现了Mimblewimble的系统虽然在初始同步时所需的数据量有所减少，
但是一旦同步完成，节点仍然会维护着一份完整的UTXO集。
其实Utreexo和Mimblewimble技术可以同时实施，这既可以减小状态大小，又可以进行高效的初始同步。

Other recent work on accumulators for blockchain state has introduced constant-size accumulators using groups of unknown order [5].
While this construction has superior properties in terms of proof size and batching, it depends on either a trusted setup or novel cryptographic primitives.
Another difficulty with accumulators using groups of unknown order is that while transmission of proofs can be batched and thus made very small, updating and maintaining many proofs (such as all of them) may be infeasible, due to proof updates not batching as is possible with Utreexo’s hash based accumulator.

还有一些有关区块链状态累加器的最新研究成果引入了使用未知顺序组的恒定大小累加器[5]。
这种构想在证明大小和批处理方面具有更好的特性，但它需要依赖可信设置或更新的密码学原语。
【译注：*可信设置（trusted setup）*又称为“可信启动”，
在密码学中是指创建一个密码学系统时生成其初始参数所需的配置。
生成的初始参数在稍后会被销毁。
之所以叫“可信设置”是因为你必须相信配置的作者一定会销毁那些参数。
具体可参见：[《What is trusted setup?》](https://zcoin.io/ufaqs/what-is-trusted-setup/)。
可信设置常用在零知识证明中。
*密码学原语*可以简单理解为：你想做什么事情？
比如“加密”、“签名”是个原语，“密钥交换”、“零知识证明”也是个原语。】
使用未知顺序组的累加器所带来的另一个困难是，
虽然可以批量发送证明从而使发送量变得很小，
但由于无法像Utreexo的基于哈希的累加器那样批量地更新证明，
因此当证明量很大时，更新和维护（例如要更新所有证明）就可能变得不可行。

# References

[1] Satoshi Nakamoto. Bitcoin: A peer-to-peer electronic cash system.

http://bitcoin.org/bitcoin.pdf, 2008.

[2] James A. Donald. Re: Bitcoin p2p e-cash paper. http: //www.metzdowd.com/pipermail/cryptography/2008-November/ 014814.html, 2008.

[3] Peter Todd. Making utxo set growth irrelevant with lowlatency delayed txo commitments. https://petertodd.org/2016/ delayed-txo-commitments, 2016. Accessed: 2019-05-10.

[4] Tom Elvis Jedusor. Mimblewimble. http://diyhpl.us/ ~ bryan/ papers2/bitcoin/mimblewimble.txt, 2016. Accessed: 2019-05-22.

[5] Dan Boneh, Benedikt Bnz, and Ben Fisch. Batching techniques for accumulators with applications to iops and stateless blockchains. Cryptology ePrint Archive, Report 2018/1188, 2018. https://eprint.iacr. org/2018/1188.
