7

Performance Evaluation

This section evaluates the performance of our system using two benchmarks: a micro-benchmark and the Andrew benchmark [15]. The micro-benchmark provides a service-independent evaluation of the performance of the replication library; it measures the latency to invoke a null operation, i.e., an operation that does nothing.

The Andrew benchmark is used to compare BFS with two other ﬁle systems: one is the NFS V2 implementation in Digital Unix, and the other is identical to BFS except without replication. The ﬁrst comparison demonstrates that our system is practical by showing that its latency is similar to the latency of a commercial system that is used daily by many users. The second comparison allows us to evaluate the overhead of our algorithm accurately within an implementation of a real service.

7.1 Experimental Setup The experiments measure normal-case behavior (i.e., there are no view changes), because this is the behavior

10 that determines the performance of the system. All experiments ran with one client running two relay processes, and four replicas. Four replicas can tolerate one Byzantine fault; we expect this reliability level to sufﬁce for most applications. The replicas and the client ran on identical DEC3000/400 Alpha workstations. These workstations have a 133 MHz Alpha 21064 processor, 128 MB of memory, and run Digital Unix version 4.0. The ﬁle system was stored by each replica on a DEC RZ26 disk. All the workstations were connected by a 10Mbit/s switched Ethernet and had DEC LANCE Ethernet interfaces. The switch was a DEC EtherWORKS 8T/TX. The experiments were run on an isolated network.

The interval between checkpoints was 128 requests, which causes garbage collection to occur several times in any of the experiments. The maximum sequence number accepted by replicas in pre-prepare messages was 256 plus the sequence number of the last stable checkpoint.

7.2 Micro-Benchmark The micro-benchmark measures the latency to invoke a null operation. It evaluates the performance of two implementations of a simple service with no state that implements null operations with arguments and results of different sizes. The ﬁrst implementation is replicated using our library and the second is unreplicated and uses UDP directly. Table 1 reports the response times measured at the client for both read-only and read- write operations. They were obtained by timing 10,000 operation invocations in three separate runs and we report the median value of the three runs. The maximum deviation from the median was always below 0.3% of the reported value. We denote each operation by a/b, where a and b are the sizes of the operation argument and result in KBytes.

arg./res. (KB) 0/0 4/0 0/4

replicated read-write read-only

without replication

3.35 (309%) 1.62 (98%)

0.82

14.19 (207%) 6.98 (51%)

4.62

8.01 (72%) 5.94 (27%)

4.66

Table 1: Micro-benchmark results (in milliseconds); the percentage overhead is relative to the unreplicated case.

The overhead introduced by the replication library is due to extra computation and communication. For exam- ple, the computation overhead for the read-write 0/0 op- eration is approximately 1.06ms, which includes 0.55ms spent executing cryptographic operations. The remain- ing 1.47ms of overhead are due to extra communication; the replication library introduces an extra message round- trip, it sends larger messages, and it increases the number of messages received by each node relative to the service without replication.

The overhead for read-only operations is signiﬁcantly lower because the optimization discussed in Section 5.1 reduces both computation and communication overheads. For example, the computation overhead for the read-only 0/0 operation is approximately 0.43ms, which includes 0.23ms spent executing cryptographic operations, and the communication overhead is only 0.37ms because the protocol to execute read-only operations uses a single round-trip.

Table 1 shows that the relative overhead is lower for the 4/0 and 0/4 operations. This is because a signiﬁcant fraction of the overhead introduced by the replication library is independent of the size of operation arguments and results. For example, in the read-write 0/4 operation, the large message (the reply) goes over the network only once (as discussed in Section 5.1) and only the cryptographic overhead to process the reply message is increased. The overhead is higher for the read-write 4/0 operation because the large message (the request) goes over the network twice and increases the cryptographic overhead for processing both request and pre-prepare messages.

It is important to note that this micro-benchmark represents the worst case overhead for our algorithm because the operations perform no work and the unreplicated server provides very weak guarantees. Most services will require stronger guarantees, e.g., authenticated connections, and the overhead introduced by our algorithm relative to a server that implementsthese guarantees will be lower. For example, the overhead of the replication library relative to a version of the unreplicated service that uses MACs for authentication is only 243% for the read-write 0/0 operation and 4% for the read-only 4/0 operation.

We can estimate a rough lower bound on the performance gain afforded by our algorithm relative to Rampart [30]. Reiter reports that Rampart has a latency of 45ms for a multi-RPC of a null message in a 10 Mbit/s Ethernet network of 4 SparcStation 10s [30]. The multi- RPC is sufﬁcient for the primary to invokea state machine operation but for an arbitrary client to invoke an operation it would be necessary to add an extra message delay and an extra RSA signature and veriﬁcation to authenticate the client; this would lead to a latency of at least 65ms (using the RSA timings reported in [29].) Even if we divide this latency by 1.7, the ratio of the SPECint92 ratings of the DEC 3000/400 and the SparcStation 10, our algorithm stillreduces the latency to invoke the read-write and read-only 0/0 operations by factors of more than 10 and 20, respectively. Note that this scaling is conservative because the network accounts for a signiﬁcant fraction of Rampart’s latency [29] and Rampart’s results were obtained using 300-bit modulus RSA signatures, which are not considered secure today unless the keys used to

11 generate them are refreshed very frequently.

There are no published performance numbers for SecureRing [16] but it would be slower than Rampart because its algorithm has more message delays and signature operations in the critical path.

7.3 Andrew Benchmark The Andrew benchmark [15] emulates a software development workload. It has ﬁve phases: (1) creates subdirectories recursively; (2) copies a source tree; (3) examines the status of all the ﬁles in the tree without examining their data; (4) examines every byte of data in all the ﬁles; and (5) compiles and links the ﬁles.

We use the Andrew benchmark to compare BFS with two other ﬁle system conﬁgurations: NFS-std, which is the NFS V2 implementation in Digital Unix, and BFS-nr, which is identical to BFS but with no replication. BFS-nr ran two simple UDP relays on the client, and on the server it ran a thin veneer linked with a version of snfsd from which all the checkpoint management code was removed. This conﬁguration does not write modiﬁed ﬁle system state to disk before replying to the client. Therefore, it does not implement NFS V2 protocol semantics, whereas both BFS and NFS-std do.

Out of the 18 operations in the NFS V2 protocol only getattr is read-only because the time-last-accessed attribute of ﬁles and directories is set by operations that would otherwise be read-only, e.g., read and lookup. The result is that our optimization for read- only operations can rarely be used. To show the impact of this optimization, we also ran the Andrew benchmark on a second version of BFS that modiﬁes the lookup operation to be read-only. This modiﬁcation violates strict Unix ﬁle system semantics but is unlikely to have adverse effects in practice.

For all conﬁgurations, the actual benchmark code ran at the client workstation using the standard NFS client implementation in the Digital Unix kernel with the same mount options. The most relevant of these options for the benchmark are: UDP transport, 4096-byte read and write buffers, allowing asynchronous client writes, and allowing attribute caching.

We report the mean of 10 runs of the benchmark for each conﬁguration. The sample standard deviation for the total time to run the benchmark was always below

2.6% of the reported value but it was as high as 14% for the individual times of the ﬁrst four phases. This high variance was also present in the NFS-std conﬁguration. The estimated error for the reported mean was below

4.5% for the individual phases and 0.8% for the total.

Table 2 shows the results for BFS and BFS-nr. The comparison between BFS-strict and BFS-nr shows that the overhead of Byzantine fault tolerance for this service is low — BFS-strict takes only 26% more time to run

BFS

phase 1 2 3 4 5 total

strict

0.55 (57%)

9.24 (82%)

7.24 (18%)

8.77 (18%)

38.68 (20%)

64.48 (26%)

r/o lookup

0.47 (34%)

7.91 (56%)

6.45 (6%)

7.87 (6%)

38.38 (19%)

61.07 (20%)

BFS-nr

0.35

5.08

6.11

7.41

32.12

51.07

Table 2: Andrew benchmark: BFS vs BFS-nr. The times are in seconds.

the complete benchmark. The overhead is lower than what was observed for the micro-benchmarks because the client spends a signiﬁcant fraction of the elapsed time computing between operations, i.e., between receiving the reply to an operation and issuing the next request, and operations at the server perform some computation. But the overhead is not uniform across the benchmark phases. The main reason for this is a variation in the amount of time the client spends computing between operations; the ﬁrst two phases have a higher relative overhead because the client spends approximately 40% of the total time computing between operations, whereas it spends approximately 70% during the last three phases.

The table shows that applying the read-only optimiza- tion to lookup improves the performance of BFS sig- niﬁcantly and reduces the overhead relative to BFS-nr to 20%. This optimization has a signiﬁcant impact in the ﬁrst four phases because the time spent waiting for lookup operations to complete in BFS-strict is at least 20% of the elapsed time for these phases, whereas it is less than 5% of the elapsed time for the last phase.

BFS

phase 1 2 3 4 5 total

strict

0.55 (-69%)

9.24 (-2%)

7.24 (35%)

8.77 (32%)

38.68 (-2%)

64.48 (3%)

r/o lookup

0.47 (-73%)

7.91 (-16%)

6.45 (20%)

7.87 (19%)

38.38 (-2%)

61.07 (-2%)

NFS-std

1.75

9.46

5.36

6.60

39.35

62.52

Table 3: Andrew benchmark: BFS vs NFS-std. times are in seconds.

The

Table 3 shows the results for BFS vs NFS-std. These results show that BFS can be used in practice — BFS- strict takes only 3% more time to run the complete benchmark. Thus, one could replace the NFS V2 implementation in Digital Unix, which is used daily by many users, by BFS without affecting the latency perceived by those users. Furthermore, BFS with the read-only optimization for the lookup operation is actually 2% faster than NFS-std.

The overhead of BFS relative to NFS-std is not the

12 same for all phases. Both versions of BFS are faster than NFS-std for phases 1, 2, and 5 but slower for the other phases. This is because during phases 1, 2, and 5 a large fraction (between 21% and 40%) of the operations issued by the client are synchronous, i.e., operations that require the NFS implementation to ensure stability of modiﬁed ﬁle system state before replying to the client. NFS-std achieves stability by writing modiﬁed state to disk whereas BFS achieves stability with lower latency using replication (as in Harp [20]). NFS-std is faster than BFS (and BFS-nr) in phases 3 and 4 because the client issues no synchronous operations during these phases.
