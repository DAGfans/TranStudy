> Source: https://eprint.iacr.org/2019/611.pdf
# 2 Related Work
# 2 相关工作

The limitations of having every client store the UTXO set have been clear since Bitcoin was ﬁrst introduced; in fact this was the ﬁrst comment ever recorded about Bitcoin [2].
Earlier work has focused on UTXO commitments [3], where some representation of the UTXO set would be included in the blockchain itself.
With a consensus rule enforcing the validity of this UTXO commitment, users could get some assurance, similar to SPV security, about the representation of the UTXO set without downloading or validating it themselves.
In contrast to UTXO commitments, Utreexo state is not transmitted to clients, but instead computed by them.

自从首次引入比特币以来，让每个客户端存储UTXO集的局限性就很明显了。实际上，这是有史以来关于比特币的第一个评论[2]。
早期的工作集中在UTXO提交[3]上，其中UTXO集以某种表示形式将包含在区块链本身中。
有了强制执行此UTXO提交有效性的共识规则，用户可以像SPV安全性一样，获得有关UTXO集表示形式的某种保证，而无需自己下载或验证它。
与UTXO提交相反，Utreexo状态不会传输给客户端，而是由客户端计算。

Techniques for speeding up or skipping parts of Initial Block Download have been proposed, the most promising of which is Mimblewimble [4].
While Mimblewimble maintains the same cryptographic security as downloading and verifying the transaction history, it is a substantial change in transaction format and is diﬃcult to implement in a backwards-compatible way to the existing Bitcoin network.
Systems which do implement Mimblewimble beneﬁt from a decrease in data needed to perform initial synchronization, but once synchronized still maintain a full UTXO set.
It’s possible that Utreexo and Mimblewimble techniques could be implemented simultaneously, which would allow for both reduced state size and eﬃcient initial synchronization.

已经提出了的加快或跳过“初始块下载”部分的技术，其中最有代表性的是Mimblewimble [4]。
尽管Mimblewimble保持与下载和验证交易历史记录相同的密码安全性，但交易格式产生了重大变化，并且难以以向后兼容的方式实现到现有比特币网络。
确实实现了Mimblewimble的系统受益于执行初始同步所需的数据减少，但是一旦同步，仍然会保留完整的UTXO集。
Utreexo和Mimblewimble技术可以同时实施，这既可以减小状态大小，又可以进行高效的初始同步。

Other recent work on accumulators for blockchain state has introduced constant-size accumulators using groups of unknown order [5].
While this construction has superior properties in terms of proof size and batching, it depends on either a trusted setup or novel cryptographic primitives.
Another diﬃculty with accumulators using groups of unknown order is that while transmission of proofs can be batched and thus made very small, updating and maintaining many proofs (such as all of them) may be infeasible, due to proof updates not batching as is possible with Utreexo’s hash based accumulator.

其他有关区块链状态累加器的最新工作已经引入了使用未知顺序组的恒定大小累加器[5]。
尽管此构造在证明大小和批量方面具有优越的属性，但它依赖于可信的设置或新颖的加密框架。
使用累加器使用未知顺序的组的另一个困难是，虽然可以分批发送证明，因此可以使发送的样本量很小，但由于无法进行类似Utreexo的基于散列累加器那样分批的证明更新，因此更新和维护许多证明（例如所有证明）可能不可行。 。

# References

[1] Satoshi Nakamoto. Bitcoin: A peer-to-peer electronic cash system.

http://bitcoin.org/bitcoin.pdf, 2008.

[2] James A. Donald. Re: Bitcoin p2p e-cash paper. http: //www.metzdowd.com/pipermail/cryptography/2008-November/ 014814.html, 2008.

[3] Peter Todd. Making utxo set growth irrelevant with lowlatency delayed txo commitments. https://petertodd.org/2016/ delayed-txo-commitments, 2016. Accessed: 2019-05-10.

[4] Tom Elvis Jedusor. Mimblewimble. http://diyhpl.us/ ~ bryan/ papers2/bitcoin/mimblewimble.txt, 2016. Accessed: 2019-05-22.

[5] Dan Boneh, Benedikt Bnz, and Ben Fisch. Batching techniques for accumulators with applications to iops and stateless blockchains. Cryptology ePrint Archive, Report 2018/1188, 2018. https://eprint.iacr. org/2018/1188.