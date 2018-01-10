Origin: https://arxiv.org/pdf/1707.01873.pdf  
Author: [Christian Cachin](https://arxiv.org/find/cs/1/au:+Cachin_C/0/1/0/all/0/1), [Marko Vukolić](https://arxiv.org/find/cs/1/au:+Vukolic_M/0/1/0/all/0/1)

Blockchain Consensus Protocols in the Wild

Christian Cachin Marko Vukoli´c IBM Research - Zurich (cca|mvu)@zurich.ibm.com 2017-07-07

Abstract

A blockchain is a distributed ledger for recording transactions, maintained by many nodes without central authority through a distributed cryptographic protocol. All nodes validate the information to be appended to the blockchain, and a consensus protocol ensures that the nodes agree on a unique order in which entries are appended. Consensus protocols for tolerating Byzantine faults have re- ceived renewed attention because they also address blockchain systems. This work discusses the process of assessing and gaining conﬁdence in the resilience of a consensus protocols exposed to faults and adversarial nodes. We advocate to follow the established practice in cryptography and computer security, relying on public reviews, detailed models, and formal proofs; the designers of several practical systems appear to be unaware of this. Moreover, we review the consensus proto- cols in some prominent permissioned blockchain platforms with respect to their fault models and resilience against attacks. The protocol comparison covers Hyperledger Fabric, Tendermint, Sym- biont, R3 Corda, Iroha, Kadena, Chain, Quorum, MultiChain, Sawtooth Lake, Ripple, Stellar, and IOTA.

1

Introduction

Blockchains or distributed ledgers are systems that provide a trustworthy service to a group of nodes or parties that do not fully trust each other. They stand in the tradition of distributed protocols for secure multiparty computation in cryptography and replicated services tolerating Byzantine faults in distributed systems. Blockchains also contain many elements from cryptocurrencies, although a blockchain system can be conceived without a currency or value tokens. Generally, the blockchain acts as a trusted and dependable third party, for maintaining shared state, mediating exchanges, and providing a secure com- puting engine. Many blockchains can execute arbitrary tasks, typically called smart contracts, written in a domain-speciﬁc or a general-purpose programming language.

In a permissionless blockchain, such as Bitcoin or Ethereum, anyone can be a user or run a node, anyone can “write” to the shared state through invoking transactions (provided transaction fees are paid for), and anyone can participate in the consensus process for determining the “valid” state. A permis- sioned blockchain in contrast, is operated by known entities, such as in consortium blockchains, where members of a consortium or stakeholders in a given business context operate a permissioned blockchain network. Permissioned blockchains systems have means to identify the nodes that can control and update the shared state, and often also have ways to control who can issue transactions. A private blockchain is a special permissioned blockchain operated by one entity, i.e., within one single trust domain.

Permissioned blockchains address many of the problems that have been studied in the ﬁeld of dis- tributed computing over decades, most prominently for developing Byzantine fault-tolerant (BFT) sys- tems. Such blockchains can beneﬁt from many techniques developed for reaching consensus, replicating state, broadcasting transactions and more, in environments where network connectivity is uncertain, nodes may crash or become subverted by an adversary, and interactions among nodes are inherently asynchronous. The wide-spread interest in blockchain technologies has triggered new research on prac- tical distributed consensus protocols. There is also a growing number of startups, programmers, and

1

arXiv:1707.01873v2 [cs.DC]

7 Jul

2017 industry groups developing blockchain protocols based on their own ideas, not relying on established knowledge.

The purpose of this paper is to give an overview of consensus protocols actually being used in the context of permissioned blockchains, to review the underlying principles, and to compare the resilience and trustworthiness of some protocols. We leave out permissionless (or “public”) blockchains that are coupled to a cryptocurrency and their consensus protocols, such as proof-of-work or proof-of-stake, al- though this is a very interesting subject by itself.

We start by pointing out that developing consensus protocols is difﬁcult and should not be undertaken in an ad-hoc manner (Sec. 2). A resilient consensus protocol is only useful when it continues to deliver the intended service under a wide range of adversarial inﬂuence on the nodes and the network. Detailed analysis and formal argumentation are necessary to gain conﬁdence that a protocol achieves its goal. In that sense, distributed computing protocols resemble cryptosystems and other security mechanisms; they require broad agreement on the underlying assumptions, detailed security models, formal reasoning, and wide-spread public discussion. Any claim for a “superior” consensus protocol that does not come with the necessary formal justiﬁcation should be dismissed, analogously to the approach of “security by obscurity,” which is universally rejected by experts.

The text continues in Section 3 with a brief review of consensus in the context of blockchains. As an example for the principle of scientiﬁc study, some serious ﬂaws in a BFT consensus protocol called Tangaroa (also known as BFT-Raft) are shown. This protocol has gained quite some popularity among proponents of permissioned blockchains but it is not implemented in any blockchain platform.

In Section 4 the consensus protocols in a number of permissioned blockchain platforms are com- pared, based on the available product descriptions or source code. Section 5 discusses the consensus mechanisms of blockchain platforms not directly following the BFT approach: Sawtooth Lake, Ripple and Stellar, and the IOTA Tangle. A summary concludes the paper.

2

Trust in a blockchain protocol

A blockchain or a distributed ledger protocol can be summarized as a secure distributed protocol that achieves certain task satisfying targeted consistency (typically atomicity or total order) and liveness (or availability) semantics. However, verifying and establishing trust in the security, consistency and live- ness semantics of a blockchain protocol is challenging. This section argues that developing consensus protocols is similar to building cryptographic systems and that it should beneﬁt from the experience and practice in cryptography.

It is generally difﬁcult to evaluate any security mechanism. First and foremost, a “secure” solution should not interfere “too much” with the functionality, i.e., primary task one tries to accomplish. But more importantly, the security tool should ensure that one can accomplish this task in a way that is resilient to problems caused by the (adversarial) environment, by preventing, deterring, withstanding, or tolerating any inﬂuence that could hinder one from accomplishing the task.

Showing that the solution works in the absence of problems and attacks is easy. The task is achieved, and output is straightforward to verify. Assessing the security is the hard part. A security solution should come with a clearly stated security model and trust assumption, under which the solution should satisfy its goal. This is widely accepted today; it prompts the question of how to validate that the solution satisﬁes its goal.

Yet, the experimental validation of a security solution in the information technology space fails very often because no experiment can exhaustively test the solution in all scenarios permitted by the model. In a way, experimentation can only demonstrate the failure of a security mechanism.

Therefore one needs to apply mathematical reasoning and formal tools to reason why the solution would remain secure under any scenario permitted by the stated trust assumption. Without such reason- ing, security claims remain vague.

In the domain of blockchain protocols, one can learn a lot from the history of cryptography. Already since the 19th century, Kerckhoffs’ principle has been widely accepted, which states that “a cryptosystem

2 should be secure even if everything about the system, except the key, is public knowledge.” It implies that any security claim of the kind that a system embodies a superior but otherwise undisclosed design should be dismissed immediately.

Starting from the pioneering work in the 1980s, modern cryptography has developed formal treat- ment, security notions, and corresponding provably secure protocols. Cryptography research has con- centrated on mathematically formalizing (a small number of) security assumptions, such as “computing discrete logarithms in particular groups is hard,” and on building complex systems and protocols that rely on these assumptions, without introducing any additional insecurity. In other words, in a “provably secure” solution, an attack on the stated goal of the solution can be turned efﬁciently into a violation of some underlying assumption.

For assessing whether the formal models are appropriate and whether the security assumptions cover the situation encountered during deployment, human judgment is needed, best exerted through careful review, study, validation, and expert agreement. The AES block cipher, for instance, was selected in 2000 by the U.S. NIST after a multi-year public review process during which many candidates were debated and assessed openly by the world-wide cryptographic research community.

During the internet boom in the late 1990s there were many claims of new and “unbreakable” cryp- tosystems, all lacking substantiation. Many of them were covered by Schneier in blog posts about “snake oil,” alluding to the history of medicine before regulation [64]:

The problem with bad security is that it looks just like good security. You can’t tell the difference by looking at the ﬁnished product. Both make the same security claims; both have the same functionality. (. . . ) Both might use the same protocols, implement the same standards, and have been endorsed by the same industry groups. Yet one is secure and the other is insecure.

Expert judgment, formal reasoning, experience, public discussion, and open validation are needed for accepting a cryptosystem as secure.

A similar development has taken place with building resilient distributed systems, whose goal is to deliver a service while facing network outages, communication failures, timing uncertainty, power loss and more. The Chubby database of Google [24] and Yahoo!’s ZooKeeper [38], developed for syn- chronizing critical conﬁguration information across data centers, support strong consistency and high availability through redundancy and tolerate benign failures and network outages. Those systems started from well-understood, mathematically speciﬁed, and formally veriﬁed protocols in the research litera- ture (e.g., Lamport’s Paxos protocol [42]). Yet it has taken considerable effort during development and testing and frequent exercising of failure scenarios during deployment [10] to achieve the desired level of resilience in practice.

Over the recent years countless proposals for new features in distributed ledger systems and com- pletely new blockchain protocols have appeared, mainly originating from the nascent ﬁntech industry and startup scene. Most of them come without formal expression of their trust assumption and security model. There is no agreed consensus in the industry on which assumptions are realistic for the intended applications, not to mention any kind of accepted standard or validation for protocols. The ﬁeld of block- chain protocols is in its infancy today, but already appears at the peak of overstated expectations [32]. Many fantastic and bold claims are made in the ﬁntech and blockchain space by startups, established companies, researchers, and self-proclaimed experts alike. This creates excitement but also confusion in the public opinion.

Broad agreement on trust assumptions, security models, formal reasoning methods, and protocol goals is needed. Developers, investors, and users in the industry should look towards the established scientiﬁc methodology in cryptography and security with building trustworthy systems, before they en- trust ﬁnancial value to new protocols. Open discussion, expert reviews, broad validation, and standards recommendations should take over and replace the hype.

3 4.10 MultiChain

The MultiChain platform (https://github.com/MultiChain/multichain) is intended for permissioned blockchains in the ﬁnancial industry and for multi-currency exchanges in a consortium, aiming at compatibility with the Bitcoin ecosystem as much as possible.

MultiChain uses a dynamic permissioned model [34]: There is a list of permitted nodes in the net- work at all times, identiﬁed by their public keys. The list can be changed through transactions executed on the blockchain, but at all times, only nodes on this list validate blocks and participate in the protocol.

As the MultiChain platform is derived from Bitcoin, its consensus mechanism is called “mining” [34]; however, in the permissioned model, the nodes do not solve computational puzzles. Instead, any permit- ted node may generate new blocks after waiting for a random timeout, subject to a diversity parameter ρ ∈ [0, 1] that constrains the acceptable miners for a given block height. More precisely, if the permitted list has length L, then a block proposal from a node is only accepted if the blockchain held by the vali- dating node does not already contain a block generated by the same node among the ⌈ρL⌉ most recent blocks. Any participating node will extend its blockchain with the ﬁrst valid block of this kind that it receives, and if it learns about different, conﬂicting chain extensions, it will select the longer one (as in Bitcoin). Furthermore, a well-behaved node will not generate a new block if its own chain already contains a block of his within the last ⌈ρL⌉ blocks.

It appears that the random timeouts and network uncertainty easily lead to forks in the ledger, even if all nodes are correct. If two different nodes may generate a valid block at roughly the same time, and any other node will append the one of which it hears ﬁrst to its chain, then these two nodes will be forked. This is not different from consensus in Bitcoin and will eventually converge to a single chain if all nodes follow the protocol. However, if a single attacking node generates transactions and blocks as it wants, and assuming that the network behaves favorably for the attack, the node can take over the entire network and revert arbitrarily many past transactions (in the same way as a “51%-attack” in Bitcoin).

Hence, MultiChain exhibits non-ﬁnal transactions similar to any proof-of-work consensus. But whereas lack of ﬁnality appears to be a consequence of the public nature of proof-of-work, and since MultiChain is permissioned, forks and non-ﬁnal decisions could be avoided here completely. The tra- ditional consensus protocols for this model, discussed in Sections 3.2 and 3.3, all reach consensus with ﬁnality. In the model of non-ﬁnal consensus decisions, with the corresponding delays and throughput constraints, the MultiChain consensus protocol can only remain consistent and live with one single cor- rect node.

Safety* Liveness

Generic nodes n n

Any t nodes crash t<n t<n

Any f nodes subverted − −

Table 12: Resilience of MultiChain consensus. Safety* denotes the non-ﬁnal consistency notion, as achieved by proof-of-work consensus, and should be understood formally as in recent work on the sub- ject [33].

4.11 Further platforms

Another extension of the Ethereum platform is HydraChain (https://github.com/HydraChain/ hydrachain/blob/develop/README.md), which adds support for creating a permissioned dis- tributed ledger using the Ethereum infrastructure. The repository describes a proprietary consensus pro- tocol “initially inspired by Tendermint.” Without clear explanation of the protocol and formal review of its properties, its correctness remains unclear.

The Swirlds hashgraph algorithm is built into a proprietary “distributed consensus platform” (https: //www.swirlds.com); a white paper is available [7] and the protocol is also implemented in an open-source consensus platform for distributed applications, called Babble (https://github.com/

16 babbleio/babble). It targets consensus for a permissioned blockchain with n nodes and f < n/3 Byzantine faults among them, i.e., the standard Byzantine consensus problem according to Section 3.3. In contrast to PBFT and other protocols discussed there, it operates in a “completely asynchronous” model. The white paper states arguments for the safety and liveness of the protocol and explains that hashgraph consensus is randomized to circumvent the FLP impossibility [31]. Since the algorithm is guaranteed to reach agreement on a binary decision (i.e., with only 0/1 outcomes) only with exponen- tially small probability in n [7, Thm. 5.16], it appears similar to Ben-Or-style randomized agreement [18, Sec. 5.5]. However, no independent validation or analysis of hashgraph consensus is available.

5

Permissionless blockchains

The most widely used consensus protocols in permissionless blockchains are proof-of-work and proof- of-stake, which appear always coupled to a cryptocurrency. There is a lot of research and development activity addressing them at the moment, and covering this would go beyond the scope of this text.

Instead, this section reviews some notable variants of protocols that do not rely on a strict notion of membership and therefore differ from the permissioned blockchain platforms reviewed in the last section. The protocols described here depart in other ways from the traditional consensus notions (crash-tolerant, Byzantine, and proof-of-work-style consensus). Conceptually they fall somewhere between the extremes of a BFT protocol and Nakamoto’s proof-of-work consensus.

5.1 Sawtooth Lake – Proof of Elapsed Time

The Hyperledger Sawtooth platform (https://github.com/hyperledger/sawtooth-core) provides means for running general-purpose smart contracts on a distributed ledger. It can use a permis- sioned and a public, permissionless mode. The platform also introduces a novel consensus protocol called Proof of Elapsed Time (PoET), originally contributed by Intel, which is based on the insight that proof-of-work essentially imposes a mandatory but random waiting time for leader election.

In particular, when ignoring the mining reward of Bitcoin, Nakamoto consensus lets all nodes par- ticipate in a probabilistic experiment, where each node is delayed for a random duration. Once the timer expires, the node can prove to all others in a veriﬁable way that it has executed the “waiting step” cor- rectly for extending the blockchain. The node propagates its solution to all others as quickly as possible, because only the longest chain is valid. With the correct relation between the tunable waiting-delay and the expected time for reaching every node with the new solution, this creates a stable consensus protocol. The Bitcoin network’s operation and mathematical analysis [33] demonstrate this.

PoET consensus executes the waiting step in a trusted hardware module, the Intel Software Guard Extensions (SGX) available in many modern Intel CPUs. Every node essentially calls an enclave inside SGX for generating a random delay, waiting accordingly, and then declaring itself to be the leader in consensus and extending the blockchain. The platform creates an attestation that can be used by any node to verify that the leader correctly waited for the proper random time. Assuming the hardware module cannot be subverted, this creates the same kind of non-ﬁnal consensus as with mining in proof-of-work.

The energy waste caused by mining goes away. However, economic investment still increases the inﬂuence on the protocol because the probability of a node becoming the leader is proportional to the number of hardware modules under its control. PoET is compatible with permissionless blockchains, but only assuming an unlimited supply of trusted modules. As the protocol’s security depends on the hard- ware module potentially running on an adversarial host, the impact of attacks will have to be understood as well (e.g., SGX is susceptible to rollback attacks [12] and key extraction [66]).

In a permissioned setting, the participating modules could be authenticated and the weight of a node can be ﬁxed through this. However, with known nodes, traditional BFT consensus protocols have several advantages compared to PoET: they are more efﬁcient, do not rely on a single vendor’s hardware, and create ﬁnal decisions. Moreover, if trusted modules are available, then BFT consensus can increase the resilience to f < n/2 subverted nodes and achieve more than 70’000 tps throughput in a LAN [40].

17 5.2 Ripple and Stellar

Ripple (https://ripple.com/) and Stellar (https://www.stellar.org/) are two globally operating exchange networks with built-in cryptocurrencies; unlike Bitcoin, they do not involve mining and operate in a somewhat permissioned fashion. The Ripple protocol consensus algorithm (RPCA) and its offspring Stellar consensus protocol (SCP) depart from the traditional security assumption for consensus protocols (i.e., some f < n/3 faulty nodes) by making their trust assumptions ﬂexible [65, 52]. This means that each node would declare on its own which nodes it trusts, instead of accepting a global assumption on which node collusions the protocol tolerates. Each node designates a list of other nodes sufﬁcient to convince itself (through the unique node list in Ripple or the quorum slice of Stellar).

Ripple and Stellar each maintain one distributed ledger governed by the protocol, which records exchanges on the respective network.

Ripple. In Ripple, the process of advancing the common distributed ledger is controlled by so-called validating nodes. They periodically start to create a new ledger entry (every few seconds) and iteratively vote in rounds on its content; each node accepts a proposed ledger update if 50%, . . . , 80% (increasing by +10% per round) of the signed updates that it receives match.

Ripple’s documentation states that 4/5 · n of all n validator nodes must be correct for maintaining correctness [65]. This would correspond to tolerating f < n/5 subverted nodes in traditional BFT systems.

Furthermore, it is obvious that a minimal overlap among the convincing-sets (i.e., unique node lists) of all pairs of validating nodes is required, since otherwise they could exhibit split-brain behavior and the ledger would fork. Ripple states that the overlap should be at least 1/5 of the size of the larger list [65]. The only peer-reviewed analysis of the Ripple protocol, however, contradicts this. Armknecht et al. [3] show that when each node has ρn validators in its convincing-set, then ledger forks are ruled out only if for every two nodes, their lists contain more than 2(1 − ρ)m common nodes, where m is the size of the larger of the two lists. This implies more than 2/5 overlap with ρ = 0.8 as chosen by the Ripple network.

Currently, Ripple “provides a default and recommended list of validators operated by Ripple and third parties,” through a static conﬁguration ﬁle; by default there are ﬁve validators operated by Ripple which trust each other and no other node. (“At present, Ripple cannot recommend any validators aside from the 5 core validators run by Ripple (the company)” [60].) It appears that this list is adopted by most validating nodes in the system; consequently, trust is by far not as decentralized as advertised.

A typical consensus process creating one new ledger entry completes in less than four seconds on average. Ripple has stated a throughput of about 1000 tps on a test network [1]. Compared to several 10’000 tps achievable on traditional BFT platforms with a small group of 4–10 validators [68], this seems unnecessarily slow.

Stellar. As Stellar evolved from Ripple, it uses similar ideas and a protocol called federated Byzantine agreement [52] within the Stellar consensus protocol (SCP). Only validator nodes participate in the protocol for reaching consensus. Each validator declares its own convincing-set (called “quorum slice” and similar to Ripple’s unique node list) that must sufﬁciently overlap with the convincing-sets of other nodes for preventing forks. A node accepts a “vote” or a transaction for the ledger when a threshold of nodes in its convincing-set conﬁrm it. Examples in the documentation and the white paper [52, Fig. 3] suggest the use of hierarchical structures with different groups organized into multiple levels, where a different threshold may exist for each group (but the “threshold should be 2/3 for the top level”).

For instance, a convincing-set (i.e., a quorum slice) in a hierarchy with two levels could be like this (expressed in percent and rounded to integers):

18 67% of Groups, where Groups = {Banks, Auditors, Advisors, Friends} 51% of Banks, where

Banks = {Bank-1, Bank-2, Bank-3} 58% of Auditors, where

Auditors = {A, B, C, D, E, F, G} 51% of Advisors, where

Advisors = {1, 2, 3} 1% of Friends, where

Friends = {Alice, Bob, Charlie, . . . , Zach}

// here: 3 of 4 sub-groups // here: 2 of 3 banks // here: 5 of 7 auditors // here: 2 of 3 advisors // here: 1 of 26 friends

Similar structures have been known as Byzantine quorum systems (BQS) [49] and are well-understood. They can readily be used to build consensus protocols for BFT systems [15]. The documentation avail- able for Stellar does not relate to this literature, however.

Furthermore, it seems that for constructing one single ledger, the convincing-sets for all useful con- ﬁgurations of SCP should intersect at the top of the suggested hierarchies. This appears to introduce some amount of centralization, similar to BQS [50].

At this time, determining the similarities and differences between the quorum slices of SCP and generic BQS for Byzantine consensus remains an open problem.

5.3 IOTA Tangle

IOTA (http://iota.org/) is heralded as a “cryptocurrency without a blockchain” and creates a Hash-DAG instead, which is called the tangle [58]. All of its tokens are created at the outset. Transactions are propagated in a peer-to-peer network like in Bitcoin; each transaction transfers tokens owned by a node to others and must be signed by the owner’s key node. A transaction also includes the solution to a proof-of-work puzzle and approves two or any k ≥ 2 transactions by including a hash of them in the transaction. This creates the DAG, with an edge pointing from each conﬁrmed transaction to the new one. A weight is assigned to the transaction proportionally to the difﬁculty of the puzzle that the node has solved for producing it. The node is supposed to choose the k transactions to conﬁrm randomly, from all transactions it is aware of, and to verify the two transactions plus all transitive predecessor transactions of them. Implicitly it should also verify the transaction(s) that have earlier assigned the tokens to the node.

A node can cheat by (1) issuing an invalid transaction (double-spending), (2) including invalid pre- decessor transactions, or (3) not selecting the transactions to conﬁrm randomly. However, other nodes would intuitively not include invalid transactions produced from (1) or (2), and therefore such a transac- tion would become orphaned. In a dense graph, it is expected that most transactions of a certain age are approved by a vast majority of all newly generated transactions.

How strongly a new transaction approves its predecessor transactions depends on the weight, i.e., the computational work, that went into producing it. The stability of the system therefore rests on the assumption that the majority of the computing power among the nodes behaves correctly. Furthermore, if there are conﬂicting parts of the graph, each node will decide on its own which side to trust. This is done through a probabilistic sampling algorithm and deciding according to a statistical test. The construction of the DAG is reminiscent of Lewenberg et al.’s [46] inclusive blockchain protocols, but the details differ.

The white paper [58] and documentation claim that the tangle hash-DAG ensures a similar level of consistency as other permissionless blockchain systems. No publicly reviewed, formal analysis is available, however. Without any independent assessment of the protocol’s properties, it remains unclear how strictly the tangle emulates a notion of consensus among the nodes.

19 6

Conclusion

This paper has summarized some of the most prominent blockchain consensus protocols, focusing on permissioned systems in the sense that their participants are identiﬁed.

We have argued that developing consensus protocols is similar to engineering cryptographic systems, and that blockchain developers should look towards the established experience in cryptography, security, and the theory of distributed systems for building trustworthy systems. Otherwise, it might be danger- ous to entrust ﬁnancial value to new protocols. Open discussion, expert reviews, broad validation, and standards recommendations should be employed.

The overview of consensus protocols and their properties contributes to this effort, by establishing a common ground for formal protocol reviews and more technical comparisons. Once sufﬁciently many systems become available publicly and are widely used, it will be interesting to compare their perfor- mance through benchmarks and to observe their resilience to actual attacks or network incidents.

Acknowledgments

We thank Andreas Kind, Pedro Moreno-Sanchez, and Bj¨orn Tackmann for interesting discussions and valuable comments.

This work has been supported in part by the European Commission through the Horizon 2020 Frame- work Programme (H2020-ICT-2014-1) under grant agreements number 643964-SUPERCLOUD and 644579 ESCUDO-CLOUD and in part by the Swiss State Secretariat for Education, Research and Inno- vation (SERI) under contracts number 15.0091 and 15.0087.

References

[1] W. Anderson. Ripple consensus ledger can sustain 1000 transactions per second. Ripple Dev Blog, https://ripple.com/dev-blog/ripple-consensus-ledger-can- sustain-1000-transactions-per-second/, 2017.

[2] E. Androulaki, C. Cachin, K. Christidis, C. Murthy, B. Nguyen, and M. Vukoli´c. Next consensus architecture proposal. Hyperledger Wiki, Fabric Design Documents, avail- able at https://github.com/hyperledger/fabric/blob/master/proposals/ r1/Next-Consensus-Architecture-Proposal.md, 2016.

[3] F. Armknecht, G. O. Karame, A. Mandal, F. Youssef, and E. Zenner. Ripple: Overview and outlook.

In M. Conti, M. Schunter, and I. G. Askoxylakis, editors, Proc. Trust and Trustworthy Computing (TRUST), volume 9229 of Lecture Notes in Computer Science, pages 163–180. Springer, 2015.

[4] H. Attiya and J. Welch. Distributed Computing: Fundamentals, Simulations and Advanced Topics.

Wiley, second edition, 2004.

[5] P. Aublin, R. Guerraoui, N. Knezevic, V. Qu´ema, and M. Vukolic. The next 700 BFT protocols.

ACM Transactions on Computer Systems, 32(4):12:1–12:45, 2015.

[6] A. Avizienis, J.-C. Laprie, B. Randell, and C. Landwehr. Basic concepts and taxonomy of depend- able and secure computing. IEEE Transactions on Dependable and Secure Computing, 1(1):11–33, 2004.

[7] L. Baird. The Swirlds hashgraph consensus algorithm: Fair, fast, Byzantine fault tolerance.

Swirlds Tech Report SWIRLDS-TR-2016-01, available online, http://www.swirlds.com/ developer-resources/whitepapers/, 2016.

20 [8] A. Bessani and J. Sousa. From Byzantine consensus to BFT state machine replication: A latency- optimal transformation. In Proc. 9th European Dependable Computing Conference, pages 37–48, 2012.

[9] A. N. Bessani, J. Sousa, and E. A. P. Alchieri. State machine replication for the masses with BFT-SMaRt. In Proc. 44th International Conference on Dependable Systems and Networks, pages 355–362, 2014.

[10] B. Beyer, C. Jones, J. Petoff, and N. R. Murphy, editors. Site Reliability Engineering: How Google Runs Production Systems. O’Reilly, Sebastopol, 2016.

[11] G. Bracha. Asynchronous Byzantine agreement protocols. Information and Computation, 75:130– 143, 1987.

[12] M. Brandenburger, C. Cachin, M. Lorenz, and R. Kapitza. Rollback and forking detection for trusted execution environments using lightweight collective memory. e-print, arXiv:1701.00981 [cs.DC], 2017.

[13] E. Buchman. Tendermint: Byzantine fault tolerance in the age of blockchains. M.Sc. Thesis, University of Guelph, Canada, June 2016.

[14] E. Buchman and J. Kwon. Private discussion, 2017.

[15] C. Cachin. Distributing trust on the Internet. In Proc. International Conference on Dependable Systems and Networks (DSN-DCCS), pages 183–192, 2001.

[16] C. Cachin. Yet another visit to Paxos. Research Report RZ 3754, IBM Research, Nov. 2009.

[17] C. Cachin. Architecture of the Hyperledger blockchain fabric. Workshop on Distributed Cryp- tocurrencies and Consensus Ledgers (DCCL 2016), 2016. URL: https://www.zurich.ibm. com/dccl/papers/cachin_dccl.pdf.

[18] C. Cachin, R. Guerraoui, and L. Rodrigues. Introduction to Reliable and Secure Distributed Pro- gramming (Second Edition). Springer, 2011.

[19] C. Cachin, K. Kursawe, F. Petzold, and V. Shoup. Secure and efﬁcient asynchronous broadcast protocols (extended abstract). In J. Kilian, editor, Advances in Cryptology: CRYPTO 2001, volume 2139 of Lecture Notes in Computer Science, pages 524–541. Springer, 2001.

[20] C. Cachin, K. Kursawe, and V. Shoup. Random oracles in Constantinople: Practical asynchronous Byzantine agreement using cryptography. Journal of Cryptology, 18(3):219–246, 2005.

[21] C. Cachin and J. A. Poritz. Secure intrusion-tolerant replication on the Internet. In Proc. Inter- national Conference on Dependable Systems and Networks (DSN-DCCS), pages 167–176, June 2002.

[22] M. Castro and B. Liskov. Practical Byzantine fault tolerance and proactive recovery. ACM Trans- actions on Computer Systems, 20(4):398–461, Nov. 2002.

[23] Chain protocol whitepaper. Available online, https://chain.com/docs/1.2/ protocol/papers/whitepaper, 2017.

[24] T. D. Chandra, R. Griesemer, and J. Redstone. Paxos made live: An engineering perspective.

In Proc. 26th ACM Symposium on Principles of Distributed Computing (PODC), pages 398–407, 2007.

[25] B. Charron-Bost, F. Pedone, and A. Schiper, editors. Replication: Theory and Practice, volume 5959 of Lecture Notes in Computer Science. Springer, 2010.

21 [26] G. V. Chockler, I. Keidar, and R. Vitenberg. Group communication speciﬁcations: A comprehensive study. ACM Computing Surveys, 33(4):427–469, 2001.

[27] A. Clement, E. L. Wong, L. Alvisi, M. Dahlin, and M. Marchetti. Making Byzantine fault tolerant systems tolerate Byzantine faults. In Proc. 6th Symp. Networked Systems Design and Implementa- tion (NSDI), pages 153–168, 2009.

[28] C. Copeland and H. Zhong. Tangaroa: A Byzantine fault tolerant raft. Class project in Distributed Systems, Stanford University, http://www.scs.stanford.edu/14au-cs244b/labs/ projects/copeland_zhong.pdf, Dec. 2014.

[29] S. Duan, H. Meling, S. Peisert, and H. Zhang. BChain: Byzantine replication with high throughput and embedded reconﬁguration. In M. K. Aguilera, L. Querzoni, and M. Shapiro, editors, Proc. 18th Conference on Principles of Distributed Systems (OPODIS), volume 8878 of Lecture Notes in Computer Science, pages 91–106. Springer, 2014.

[30] C. Dwork, N. Lynch, and L. Stockmeyer. Consensus in the presence of partial synchrony. Journal of the ACM, 35(2):288–323, 1988.

[31] M. J. Fischer, N. A. Lynch, and M. S. Paterson. Impossibility of distributed consensus with one faulty process. Journal of the ACM, 32(2):374–382, Apr. 1985.

[32] D. Furlonger and R. Valdes. Hype cycle for blockchain technologies and the programmable economy, 2016. http://www.gartner.com/smarterwithgartner/3-trends- appear-in-the-gartner-hype-cycle-for-emerging-technologies-2016, July 2016.

[33] J. A. Garay, A. Kiayias, and N. Leonardos. The bitcoin backbone protocol: Analysis and appli- cations. In Advances in Cryptology: Eurocrypt 2015, volume 9057 of Lecture Notes in Computer Science, pages 281–310. Springer, 2015.

[34] G. Greenspan. Multichain private blockchain — white paper. http://www.multichain.

com/download/MultiChain-White-Paper.pdf, 2016.

[35] R. Guerraoui, R. R. Levy, B. Pochon, and V. Qu´ema. Throughput optimal total order broadcast for cluster environments. ACM Transactions on Computer Systems, 28(2):5:1–5:32, 2010.

[36] V. Hadzilacos and S. Toueg. Fault-tolerant broadcasts and related problems. In S. J. Mullender, editor, Distributed Systems (2nd Ed.). ACM Press & Addison-Wesley, New York, 1993. Expanded version appears as Technical Report TR94-1425, Department of Computer Science, Cornell Uni- versity, Ithaca NY, 1994.

[37] M. Hearn. Corda: A distributed ledger. Available online, https://docs.corda.net/ _static/corda-technical-whitepaper.pdf, 2016.

[38] P. Hunt, M. Konar, F. P. Junqueira, and B. Reed. ZooKeeper: Wait-free coordination for internet- scale systems. In Proc. USENIX Annual Technical Conference, 2010.

[39] F. Junqueira, B. Reed, and M. Seraﬁni. Zab: High-performance broadcast for primary-backup systems. In Proc. 41st International Conference on Dependable Systems and Networks, 2011.

[40] R. Kapitza, J. Behl, C. Cachin, T. Distler, S. Kuhnle, S. V. Mohammadi, W. Schr¨oder-Preikschat, and K. Stengel. CheapBFT: Resource-efﬁcient Byzantine fault tolerance. In Proc. 7th European Conference on Computer Systems (EuroSys), pages 295–308, Apr. 2012.

[41] M. Kleppmann. Designing Data-Intensive Applications. O’Reilly Media, 2017.

22 [42] L. Lamport. The part-time parliament. ACM Transactions on Computer Systems, 16(2):133–169, May 1998.

[43] L. Lamport. Paxos made simple. SIGACT News, 32(4):51–58, 2001.

[44] L. Lamport, R. Shostak, and M. Pease. The Byzantine generals problem. ACM Transactions on Programming Languages and Systems, 4(3):382–401, July 1982.

[45] B. Lampson. The ABCD’s of Paxos. In Proc. 20th ACM Symposium on Principles of Distributed Computing (PODC), 2001.

[46] Y. Lewenberg, Y. Sompolinsky, and A. Zohar. Inclusive block chain protocols. In R. B¨ohme and

T. Okamoto, editors, Proc. Financial Cryptography and Data Security (FC 2015), volume 8975 of Lecture Notes in Computer Science, pages 528–547. Springer, 2015.

[47] B. Liskov. From viewstamped replication to Byzantine fault tolerance. In B. Charron-Bost, F. Pe- done, and A. Schiper, editors, Replication: Theory and Practice, volume 5959 of Lecture Notes in Computer Science, pages 121–149. Springer, 2010.

[48] B. Liskov and J. Cowling. Viewstamped replication revisited. MIT-CSAIL-TR-2012-021, July 2012.

[49] D. Malkhi and M. K. Reiter. Byzantine quorum systems. Distributed Computing, 11(4):203–213, 1998.

[50] D. Malkhi, M. K. Reiter, and A. Wool. The load and availability of Byzantine quorum systems.

SIAM Journal on Computing, 29(6):1889–1906, 2000.

[51] W. Martino. Kadena — the ﬁrst scalable, high performance private blockchain. Whitepaper, http:

//kadena.io/docs/Kadena-ConsensusWhitePaper-Aug2016.pdf, 2016.

[52] D. Mazi`eres. The Stellar consensus protocol: A federated model for Internet-level consensus.

Stellar, available online, https://www.stellar.org/papers/stellar-consensus- protocol.pdf, 2016.

[53] A. Miller, Y. Xia, K. Croman, E. Shi, and D. Song. The honey badger of BFT protocols. In Proc.

ACM Conference on Computer and Communications Security (CCS), 2016.

[54] S. Nakamoto. Bitcoin: A peer-to-peer electronic cash system. Whitepaper, 2009. http:// bitcoin.org/bitcoin.pdf.

[55] B. M. Oki and B. Liskov. Viewstamped replication: A new primary copy method to support highly- available distributed systems. In Proc. 7th ACM Symposium on Principles of Distributed Computing (PODC), pages 8–17, 1988.

[56] D. Ongaro and J. K. Ousterhout. In search of an understandable consensus algorithm. In Proc.

USENIX Annual Technical Conference, pages 305–319, 2014.

[57] M. Pease, R. Shostak, and L. Lamport. Reaching agreement in the presence of faults. Journal of the ACM, 27(2):228–234, Apr. 1980.

[58] S. Popov. The tangle. White paper, available at https://iota.org/IOTA_Whitepaper.

pdf, 2016.

[59] M. Raynal. Communication and Agreement Abstractions for Fault-Tolerant Asynchronous Dis- tributed Systems. Synthesis Lectures on Distributed Computing Theory. Morgan & Claypool, 2010.

23 [60] Ripple. Operating rippled servers. Available online, https://ripple.com/build/ rippled-setup/#properties-of-a-good-validator, 2017.

[61] G. Samman. Kadena: The ﬁrst real private blockchain. http://sammantics.com/blog/ 2016/11/29/kadena-the-first-real-private-blockchain, Nov. 2016.

[62] G. Santos Veronese, M. Correia, A. N. Bessani, and L. C. Lung. Spin one’s wheels? Byzantine fault tolerance with a spinning primary. In Proc. 28th Symposium on Reliable Distributed Systems (SRDS), pages 135–144, 2009.

[63] F. B. Schneider. Implementing fault-tolerant services using the state machine approach: A tutorial.

ACM Computing Surveys, 22(4):299–319, Dec. 1990.

[64] B. Schneier. Snake oil. https://www.schneier.com/crypto-gram/archives/ 1999/0215.html, Feb. 1999.

[65] D. Schwartz, N. Youngs, and A. Britto. The Ripple protocol consensus algorithm. Ripple Inc., avail- able online, https://ripple.com/files/ripple_consensus_whitepaper.pdf, 2017.

[66] M. Schwarz, S. Weiser, D. Gruss, C. Maurice, and S. Mangard. Malware guard extension: Using SGX to conceal cache attacks. e-print, arXiv:1702.08719 [cs.CR], 2017.

[67] A. Shraer, B. Reed, D. Malkhi, and F. P. Junqueira. Dynamic reconﬁguration of primary/backup clusters. In Proc. USENIX Annual Technical Conference, pages 425–437, 2012.

[68] J. Sousa and A. Bessani. Separating the WHEAT from the chaff: An empirical design for geo- replicated state machines. In Proc. 34th Symposium on Reliable Distributed Systems (SRDS), pages 146–155, 2015.

[69] T. K. Srikanth and S. Toueg. Simulating authenticated broadcasts to derive simple fault-tolerant algorithms. Distributed Computing, 2:80–94, 1987.

[70] T. Swanson. Consensus-as-a-service: A brief report on the emergence of permissioned, distributed ledger systems. Report, available online, Apr. 2015. URL: http://www.ofnumbers.com/ wp-content/uploads/2015/04/Permissioned-distributed-ledgers.pdf.

[71] R. van Renesse, N. Schiper, and F. B. Schneider. Vive la diff´erence: Paxos vs. viewstamped repli- cation vs. zab. IEEE Transactions on Dependable and Secure Computing, 12(4):472–484, 2015.

[72] R. van Renesse and F. B. Schneider. Chain replication for supporting high throughput and avail- ability. In Proc. 6th Symp. Operating Systems Design and Implementation (OSDI), 2004.

[73] M. Vukoli´c. Quorum Systems: With Applications to Storage and Consensus. Synthesis Lectures on Distributed Computing Theory. Morgan & Claypool, 2012.

[74] M. Vukoli´c. Rethinking permissioned blockchains. In Proc. ACM Workshop on Blockchain, Cryp- tocurrencies and Contracts (BCC ’17), 2017.
