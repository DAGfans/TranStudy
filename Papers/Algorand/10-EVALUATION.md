10

EVALUATION

Our evaluation quantitatively answers the following:

• What is the latency that Algorand can achieve for con- firming transactions, and how does it scale as the number of users grows? (§10.1)

• What throughput can Algorand achieve in terms of trans- actions per second? (§10.2)

• What are Algorand’s CPU, bandwidth, and storage costs?

(§10.3)

• How does Algorand perform when users misbehave?

(§10.4)

• Does Algorand choose reasonable timeout parameters?

(§10.5)

To answer these questions, we deploy our prototype of Algorand on Amazon’s EC2 using 1,000 m4.2xlarge virtual machines (VMs), each of which has 8 cores and up to 1 Gbps network throughput. To measure the performance of Algo- rand with a large number of users, we run multiple Algorand users (each user is a process) on the same VM. By default, we run 50 users per VM, and users propose a 1 MByte block. To simulate commodity network links, we cap the bandwidth for each Algorand process to 20 Mbps. To model network la- tency we use inter-city latency and jitter measurements [52] and assign each machine to one of 20 major cities around the world; latency within the same city is modeled as negligible. We assign an equal share of money to each user; the equal distribution of money maximizes the number of messages that users need to process. Graphs in the rest of this section plot the time it takes for Algorand to complete an entire round, and include the minimum, median, maximum, 25th, and 75th percentile times across all users.

10.1 Latency

Figure 5 shows results with the number of users varying from 5,000 to 50,000 (by varying the number of active VMs from 100 to 1,000). The results show that Algorand can confirm transactions in well under a minute, and the latency is near- constant as the number of users grows. (Sinceτ final = 10,000, the time it takes to complete the final step increases until there are 10,000 users in the system; before this point, users are selected more than once and send fewer votes with higher weights.)

Figure 5: Latency for one round of Algorand, with 5,000 to 50,000 users.
Figure 6: Latency for one round of Algorand in a configura- tion with 500 users per VM, using 100 to 1,000 VMs.

To determine if Algorand continues to scale to even more users, we run an experiment with 500 Algorand user pro- cesses per VM. This configuration runs into two bottlenecks: CPU time and bandwidth. Most of the CPU time is spent verifying signatures and VRFs. To alleviate this bottleneck in our experimental setup, for this experiment we replace verifications with sleeps of the same duration. We are un- able to alleviate the bandwidth bottleneck, since each VM’s network interface is maxed out; instead, we increase λ step to 1 minute.

Figure 6 shows the results of this experiment, scaling the number of users from 50,000 to 500,000 (by varying the num- ber of VMs from 100 to 1,000). The latency in this experiment is about 4× higher than in Figure 5, even for the same num- ber of users, owing to the bandwidth bottleneck. However, the scaling performance remains roughly flat all the way to 500,000 users, suggesting that Algorand scales well.

10.2 Throughput

In the following set of experiments we deploy 50,000 users on our 1,000 VMs (50 users per machine). Figure 7 shows the results with a varying block size. The figure breaks the Algorand round into three parts. Block proposal (§6), at the bottom of the graph, is the time it takes a user to obtain the proposed block. The block proposal time for small block sizes is dominated by the λ priority +λ stepvar wait time. For large block sizes, the time to gossip the large block contents dominates. BA⋆ except for the final step, in the middle of the graph, is the time it takes for BA⋆ to reach the final step. Finally, BA⋆’s final step, at the top of the graph, is the time it takes BA⋆ to complete the final step. We break out the final step separately because, for the purposes of through- put, it could be pipelined with the next round (although our prototype does not do so).
Figure 7: Latency for one round of Algorand as a function of the block size.
The results show that Algorand’s agreement time (i.e., BA⋆) is independent of the block size, and stays about the same (12 seconds) even for large blocks. The throughput can be further increased by pipelining the final step, which takes about 6 seconds, with the next round of Algorand. The fixed time for running BA⋆ and the linear growth in block propagation time (with the size of the block) suggest that increasing the block size allows one to amortize the time it takes to run BA⋆ to commit more data, and therefore reach a throughput that maximizes the network capability.

At its lowest latency, Algorand commits a 2 MByte block in about 22 seconds, which means it can commit 327 MBytes of transactions per hour. For comparison, Bitcoin commits a 1 MByte block every 10 minutes, which means it can com- mit 6 MBytes of transactions per hour [9]. As Algorand’s block size grows, Algorand achieves higher throughput at the cost of some increase to latency. For example, with a 10 MByte block size, Algorand commits about 750 MBytes of transactions per hour, which is 125× Bitcoin’s throughput.

10.3 Costs of running Algorand

Users running Algorand incur CPU, network, and storage costs. The CPU cost of running Algorand is modest; when running 50 users per VM, CPU usage on the 8-core VM was about 40% (most of it for verifying signatures and VRFs), meaning each Algorand process uses about 6.5% of a core. In terms of bandwidth, each user in our experiment with 1 MByte blocks and 50,000 users uses about 10 Mbit/sec (em- pirically computed as the total amount of data sent, divided by the duration of the experiment). We note that the com- munication cost per user is independent of the number of users running Algorand, since users have an expected fixed number of neighbors they gossip messages to, and the num- ber of messages in the consensus protocol depends on the committee size (rather than the total number of users).
Figure 8: Latency for one round of Algorand with a varying fraction of malicious users, out of a total of 50,000 users.
In terms of storage cost, Algorand stores block certificates in order to prove to new users that a block was committed. This storage cost is in addition to the blocks themselves. Each block certificate is 300 KBytes, independent of the block size; for 1 MByte blocks, this would be a ∼30% storage overhead. Sharding block storage across users (§8.3) reduces storage costs proportionally. For example, sharding modulo 10 would require each user to store, on average, 130 KB for every 1MB block that is appended to the ledger.

10.4 Misbehaving users

Algorand’s safety is guaranteed by BA⋆ (§7), but proving this experimentally would require testing all possible attacker strategies, which is infeasible. However, to experimentally show that our Algorand prototype handles malicious users, we choose one particular attack strategy. We force the block proposer with the highest priority to equivocate about the proposed block: namely, the proposer sends one version of the block to half of its peers, and another version to others (note that as an optimization, if a user receives to conflicting versions of a block from the highest priority block proposer before the block proposal step is complete, he discards both proposals and starts BA⋆ with the empty block). Malicious users that are chosen to be part of the BA⋆ committee vote for both blocks. Figure 8 shows how Algorand’s performance is affected by the weighted fraction of malicious users. The results show that, at least empirically for this particular at- tack, Algorand is not significantly affected.

10.5 Timeout parameters

The above results confirm that BA⋆ steps finish in well un- der λ step (20 seconds), that the difference between 25th and 75th percentiles of BA⋆ completion times is under λ stepvar (5 seconds), and that blocks are gossiped within λ block (1 minute). We separately measure the time taken to propa- gate a block proposer’s priority and proof; it is consistently around 1 second, well under λ priority (5 seconds), confirming the measurements by Decker and Wattenhofer [18].
