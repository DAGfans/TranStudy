> Source: https://eprint.iacr.org/2019/611.pdf
# 2 Related Work
# 2 相关工作

The limitations of having every client store the UTXO set have been clear since Bitcoin was ﬁrst introduced; in fact this was the ﬁrst comment ever recorded about Bitcoin [2].
Earlier work has focused on UTXO commitments [3], where some representation of the UTXO set would be included in the blockchain itself.
With a consensus rule enforcing the validity of this UTXO commitment, users could get some assurance, similar to SPV security, about the representation of the UTXO set without downloading or validating it themselves.
In contrast to UTXO commitments, Utreexo state is not transmitted to clients, but instead computed by them.

Techniques for speeding up or skipping parts of Initial Block Download have been proposed, the most promising of which is Mimblewimble [4].
While Mimblewimble maintains the same cryptographic security as downloading and verifying the transaction history, it is a substantial change in transaction format and is diﬃcult to implement in a backwards-compatible way to the existing Bitcoin network.
Systems which do implement Mimblewimble beneﬁt from a decrease in data needed to perform initial synchronization, but once synchronized still maintain a full UTXO set.
It’s possible that Utreexo and Mimblewimble techniques could be implemented simultaneously, which would allow for both reduced state size and eﬃcient initial synchronization.

Other recent work on accumulators for blockchain state has introduced constant-size accumulators using groups of unknown order [5].
While this construction has superior properties in terms of proof size and batching, it depends on either a trusted setup or novel cryptographic primitives.
Another diﬃculty with accumulators using groups of unknown order is that while transmission of proofs can be batched and thus made very small, updating and maintaining many proofs (such as all of them) may be infeasable, due to proof updates not batching as is possible with Utreexo’s hash based accumulator.

# References

[1] Satoshi Nakamoto. Bitcoin: A peer-to-peer electronic cash system.

http://bitcoin.org/bitcoin.pdf, 2008.

[2] James A. Donald. Re: Bitcoin p2p e-cash paper. http: //www.metzdowd.com/pipermail/cryptography/2008-November/ 014814.html, 2008.

[3] Peter Todd. Making utxo set growth irrelevant with lowlatency delayed txo commitments. https://petertodd.org/2016/ delayed-txo-commitments, 2016. Accessed: 2019-05-10.

[4] Tom Elvis Jedusor. Mimblewimble. http://diyhpl.us/ ~ bryan/ papers2/bitcoin/mimblewimble.txt, 2016. Accessed: 2019-05-22.

[5] Dan Boneh, Benedikt Bnz, and Ben Fisch. Batching techniques for accumulators with applications to iops and stateless blockchains. Cryptology ePrint Archive, Report 2018/1188, 2018. https://eprint.iacr. org/2018/1188.