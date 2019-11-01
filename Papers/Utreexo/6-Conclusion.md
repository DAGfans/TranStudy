> Source: https://eprint.iacr.org/2019/611.pdf
# 6 Conclusion

We have introduced a hash-based dynamic accumulator and architecture for using this accumulator in the Bitcoin network.
 Nodes using the accumulator need only store a logarithmically sized representation of the UTXO set, greatly reducing storage space and disk seek times.
 The trade-oﬀ is the additional download requirements of inclusion proofs, but with proof aggregation and caching the increase is of a manageable size.

# Acknowledgments

Thanks to Neha Narula and Cory Fields for discussion without which this paper would not have happened.
 Thanks to Pieter Wuille for discussions about the applicability of other accumulator designs and the diﬃculties of bridge nodes (as well as coining the term).
 Thanks to Sophia Yakoubov for discussing her work which can be extended and applied here.
 Thanks to Kalle Alm for discussion about caching strategies, Dan Cline for suggesting optimal caching strategies, and Peter Malamud Smith for comments and feedback.

# References
[1] Satoshi Nakamoto. Bitcoin: A peer-to-peer electronic cash system.

http://bitcoin.org/bitcoin.pdf, 2008.

[2] James A. Donald. Re: Bitcoin p2p e-cash paper. http: //www.metzdowd.com/pipermail/cryptography/2008-November/ 014814.html, 2008.

[3] Peter Todd. Making utxo set growth irrelevant with lowlatency delayed txo commitments. https://petertodd.org/2016/ delayed-txo-commitments, 2016. Accessed: 2019-05-10.

[4] Tom Elvis Jedusor. Mimblewimble. http://diyhpl.us/ ~ bryan/ papers2/bitcoin/mimblewimble.txt, 2016. Accessed: 2019-05-22.

[5] Dan Boneh, Benedikt Bnz, and Ben Fisch. Batching techniques for accumulators with applications to iops and stateless blockchains. Cryptology ePrint Archive, Report 2018/1188, 2018. https://eprint.iacr. org/2018/1188.

20 [6] Josh Benaloh and Michael de Mare. One-way accumulators: A decentralized alternative to digital sinatures (extended abstract). In EUROCRYPT, 1993.

[7] Ralph C. Merkle. Protocols for public key cryptosystems. 1980 IEEE Symposium on Security and Privacy, pages 122–122, 1980.

[8] Leonid Reyzin and Sophia Yakoubov. Eﬃcient asynchronous accumulators for distributed pki. Cryptology ePrint Archive, Report 2015/718, 2015. https://eprint.iacr.org/2015/718.

[9] Charles Bouillaguet, Pierre-Alain Fouque, Adi Shamir, and Sebastien Zimmer. Second preimage attacks on dithered hash functions. Cryptology ePrint Archive, Report 2007/395, 2007. https://eprint.iacr. org/2007/395.

[10] Laszlo A. Belady. A study of replacement algorithms for a virtual-storage computer. IBM Systems journal, 5(2):78–101, 1966.

[11] MIT DCI. Utreexo implementation. https://github.com/mit-dci/ utreexo, 2019.