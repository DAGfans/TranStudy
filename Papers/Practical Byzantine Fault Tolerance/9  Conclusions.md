9

Conclusions

This paper has described a new state-machine replication algorithm that is able to tolerate Byzantine faults and can be used in practice: it is the ﬁrst to work correctly in an asynchronous system like the Internet and it improves the performance of previous algorithms by more than an order of magnitude.

The paper also described BFS, a Byzantine-fault- tolerant implementation of NFS. BFS demonstrates that it is possible to use our algorithm to implement real services with performance close to that of an unreplicated service — the performance of BFS is only 3% worse than that of the standard NFS implementation in Digital Unix. This good performance is due to a number of important optimizations, including replacing public-key signatures by vectors of message authentication codes, reducing the size and number of messages, and the incremental checkpoint-management techniques.

One reason why Byzantine-fault-tolerant algorithms will be important in the future is that they can allow systems to continue to work correctly even when there are software errors. Not all errors are survivable; our approach cannot mask a software error that occurs

13 at all replicas. However, it can mask errors that occur independently at different replicas, including nondeterministic software errors, which are the most problematic and persistent errors since they are the hardest to detect. In fact, we encountered such a software bug while running our system, and our algorithmwas able to continue running correctly in spite of it.

There isstillmuch work to do on improvingour system. One problem of special interest is reducing the amount of resources required to implement our algorithm. The number of replicas can be reduced by using replicas as witnesses that are involved in the protocol only when some full replica fails. We also believe that it is possible to reduce the number of copies of the state to 1 but the details remain to be worked out.

Acknowledgments We would like to thank Atul Adya, Chandrasekhar Boyapati, Nancy Lynch, Sape Mullender, AndrewMyers, Liuba Shrira, and the anonymous referees for their helpful comments on drafts of this paper.

References

[1] M. Bellare and D. Micciancio. A New Paradigm for Collision- free Hashing: Incrementality at Reduced Cost. In Advances in Cryptology – Eurocrypt 97, 1997.

[2] G. Bracha and S. Toueg. Asynchronous Consensus and Broadcast Protocols. Journal of the ACM, 32(4), 1995.

[3] R. Canneti and T. Rabin. Optimal Asynchronous Byzantine Agreement. Technical Report #92-15, Computer Science Department, Hebrew University, 1992.

[4] M. Castro and B. Liskov. A Correctness Proof for a Practi- cal Byzantine-Fault-Tolerant Replication Algorithm. Technical Memo MIT/LCS/TM-590, MIT Laboratory for Computer Sci- ence, 1999.

[5] M. Castro and B. Liskov. Authenticated Byzantine Fault Tolerance Without Public-Key Cryptography. Technical Memo MIT/LCS/TM-589, MIT Laboratoryfor Computer Science, 1999.

[6] F. Cristian, H. Aghili, H. Strong, and D. Dolev. Atomic Broadcast:

From Simple Message Diffusion to Byzantine Agreement. In International Conference on Fault Tolerant Computing, 1985.

[7] S. Deering and D. Cheriton. Multicast Routing in Datagram Internetworks and Extended LANs. ACM Transactions on Computer Systems, 8(2), 1990.

[8] H. Dobbertin. The Status of MD5 After a Recent Attack. RSA Laboratories’ CryptoBytes, 2(2), 1996.

[9] M. Fischer, N. Lynch, and M. Paterson. Impossibility of Distributed Consensus With One Faulty Process. Journal of the ACM, 32(2), 1985.

[10] J. Garay and Y. Moses. Fully Polynomial Byzantine Agreement for n 3t Processors in t+1 Rounds. SIAM Journal of Computing, 27(1), 1998.

[11] D. Gawlick and D. Kinkade. Varieties of Concurrency Control in IMS/VS Fast Path. Database Engineering, 8(2), 1985.

[12] D. Gifford. Weighted Voting for Replicated Data. In Symposium on Operating Systems Principles, 1979.

[13] M. Herlihy and J. Tygar. How to make replicated data secure.

[14] M. Herlihy and J. Wing. Axioms for Concurrent Objects. In ACM Symposium on Principles of Programming Languages, 1987.

[15] J. Howard et al. Scale and performance in a distributed ﬁle system.

ACM Transactions on Computer Systems, 6(1), 1988.

[16] K. Kihlstrom, L. Moser, and P. Melliar-Smith. The SecureRing Protocols for Securing Group Communication. In Hawaii International Conference on System Sciences, 1998.

[17] L. Lamport. Time, Clocks, and the Ordering of Events in a Distributed System. Commun. ACM, 21(7), 1978.

[18] L. Lamport. The Part-Time Parliament. Technical Report 49, DEC Systems Research Center, 1989.

[19] L. Lamport, R. Shostak, and M. Pease. The Byzantine Generals Problem. ACM Transactions on Programming Languages and Systems, 4(3), 1982.

[20] B. Liskov et al. Replication in the Harp File System. In ACM Symposium on Operating System Principles, 1991.

[21] N. Lynch. Distributed Algorithms. Morgan Kaufmann Publishers, 1996.

[22] D. Malkhi and M. Reiter. A High-Throughput Secure Reliable Multicast Protocol. In Computer Security Foundations Workshop, 1996.

[23] D. Malkhi and M. Reiter. Byzantine Quorum Systems. In ACM Symposium on Theory of Computing, 1997.

[24] D. Malkhi and M. Reiter. Unreliable Intrusion Detection in Distributed Computations. In Computer Security Foundations Workshop, 1997.

[25] D. Malkhi and M. Reiter. Secure and Scalable Replication in Phalanx. In IEEE Symposium on Reliable Distributed Systems, 1998.

[26] B. Oki and B. Liskov. Viewstamped Replication: A New Primary Copy Method to Support Highly-Available Distributed Systems. In ACM Symposium on Principles of Distributed Computing, 1988.

[27] B. Preneel and P. Oorschot. MDx-MAC and Building Fast MACs from Hash Functions. In Crypto 95, 1995.

[28] C. Pu, A. Black, C. Cowan, and J. Walpole. A Specialization Toolkit to Increase the Diversity of Operating Systems. In ICMAS Workshop on Immunity-Based Systems, 1996.

[29] M. Reiter. Secure Agreement Protocols. In ACM Conference on Computer and Communication Security, 1994.

[30] M. Reiter. The Rampart Toolkit for Building High-Integrity Services. Theory and Practice in Distributed Systems (LNCS

938), 1995.

[31] M. Reiter. A Secure Group Membership Protocol. IEEE Transactions on Software Engineering, 22(1), 1996.

[32] R. Rivest. The MD5 Message-Digest Algorithm. Internet RFC- 1321, 1992.

[33] R. Rivest, A. Shamir, and L. Adleman. A Method for Obtaining Digital Signatures and Public-Key Cryptosystems. Communications of the ACM, 21(2), 1978.

[34] F. Schneider. Implementing Fault-Tolerant Services Using The State Machine Approach: A Tutorial. ACM Computing Surveys, 22(4), 1990.

[35] A. Shamir. How to share a secret. Communications of the ACM, 22(11), 1979.

[36] G. Tsudik. Message Authentication with One-Way Hash Functions. ACMComputer Communications Review, 22(5), 1992.

[37] M. Wiener. Performance Comparison of Public-Key Cryptosys- tems. RSA Laboratories’ CryptoBytes, 4(1), 1998.
