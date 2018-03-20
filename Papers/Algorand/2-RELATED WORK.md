## 2 RELATED WORKProof-of-work.
Bitcoin [ 41 ], the predominant cryptocurrency, uses proof-of-work to ensure that everyone agreeson the set of approved transactions; this approach is often called “Nakamoto consensus.
” Bitcoin must balance thelength of time to compute a new block with the possibility of wasted work [ 41 ], and sets parameters to generate anew block every 10 minutes on average.
 Nonetheless, dueto the possibility of forks, it is widely suggested that userswait for the blockchain to grow by at least six blocks before considering their transaction to be confirmed [7].
 This```means transactions in Bitcoin take on the order of an hourto be confirmed.
 Many follow-on cryptocurrencies adoptBitcoin’s proof-of-work approach and inherit its limitations.
The possibility of forks also makes it difficult for new usersto bootstrap securely: an adversary that isolates the user’snetwork can convince the user to use a particular fork of theblockchain [28].
By relying on Byzantine agreement, Algorand eliminatesthe possibility of forks, and avoids the need to reason aboutmining strategies [ 8 , 25 , 46 ].
 As a result, transactions areconfirmed on the order of a minute.
 To make the Byzantine agreement robust to Sybil attacks, Algorand associatesweights with users according to the money they hold.
 Othertechniques have been proposed in the past to resist Sybilattacks in Byzantine-agreement-based cryptocurrencies, including having participants submit security deposits andpunishing those who deviate from the protocol [13].
``````Byzantine consensus.
 Byzantine agreement protocolshave been used to replicate a service across a small groupof servers, such as in PBFT [ 15 ].
 Follow-on work has shownhow to make Byzantine fault tolerance perform well andscale to dozens of servers [ 1 , 17 , 33 ].
 One downside of Byzantine fault tolerance protocols used in this setting is that theyrequire a fixed set of servers to be determined ahead of time;allowing anyone to join the set of servers would open up theprotocols to Sybil attacks.
 These protocols also do not scaleto the large number of users targeted by Algorand.
BA⋆is aByzantine consensus protocol that does not rely on a fixedset of servers, which avoids the possibility of targeted attackson well-known servers.
 By weighing users according to theircurrency balance,BA⋆allows users to join the cryptocurrency without risking Sybil attacks, as long as the fraction ofthe money held by honest users is at least a constant greaterthan 2/3.
BA⋆’s design also allows it to scale to many users(e.
g.
, 500,000 shown in our evaluation) using VRFs to fairlyselect a random committee.
Most Byzantine consensus protocols require more than2 / 3 of servers to be honest, and Algorand’sBA⋆inheritsthis limitation (in the form of 2 / 3 of the money being heldby honest users).
 BFT2F [ 35 ] shows that it is possible toachieve “fork∗-consensus” with just over half of the serversbeing honest, but fork∗-consensus would allow an adversary to double-spend on the two forked blockchains, whichAlgorand avoids.
Honey Badger [ 39 ] demonstrated how Byzantine fault tolerance can be used to build a cryptocurrency.
 Specifically,Honey Badger designates a set of servers to be in chargeof reaching consensus on the set of approved transactions.
This allows Honey Badger to reach consensus within 5 minutes and achieve a throughput of 200 KBytes/sec of dataappended to the ledger using 10 MByte blocks and 104 participating servers.
 One downside of this design is that thecryptocurrency is no longer decentralized; there are a fixedset of servers chosen when the system is first configured.
Fixed servers are also problematic in terms of targeted at```tacks that either compromise the servers or disconnect themfrom the network.
 Algorand achieves better performance(confirming transactions in about a minute, reaching similarthroughput) without having to choose a fixed set of serversahead of time.
Bitcoin-NG [ 26 ] suggests using the Nakamoto consensusto elect a leader, and then have that leader publish blocksof transactions, resulting in an order of magnitude of improvement in latency of confirming transactions over Bitcoin.
Hybrid consensus [ 30 , 32 , 42 ] refines the approach of usingthe Nakamoto consensus to periodically select a group ofparticipants (e.
g.
, every day), and runs a Byzantine agreement between selected participants to confirm transactionsuntil new servers are selected.
 This allows improving performance over standard Nakamoto consensus (e.
g.
, Bitcoin); forexample, ByzCoin [ 32 ] provides a latency of about 35 seconds and a throughput of 230 KBytes/sec of data appended tothe ledger with an 8 MByte block size and 1000 participantsin the Byzantine agreement.
 Although Hybrid consensusmakes the set of Byzantine servers dynamic, it opens up thepossibility of forks, due to the use of proof-of-work consensus to agree on the set of servers; this problem cannot arisein Algorand.
Pass and Shi’s paper [ 42 ] acknowledges that the Hybridconsensus design is secure only with respect to a “mildlyadaptive” adversary that cannot compromise the selectedservers within a day (the participant selection interval), andexplicitly calls out the open problem of handling fully adaptive adversaries.
 Algorand’sBA⋆explicitly addresses thisopen problem by immediately replacing any chosen committee members.
 As a result, Algorand is not susceptible toeither targeted compromises or targeted DoS attacks.
Stellar [ 36 ] takes an alternative approach to using Byzantine consensus in a cryptocurrency, where each user can trustquorums of other users, forming a trust hierarchy.
 Consistency is ensured as long as all transactions share at least onetransitively trusted quorum of users, and sufficiently manyof these users are honest.
 Algorand avoids this assumption,which means that users do not have to make complex trustdecisions when configuring their client software.
Proof of stake.
Algorand assigns weights to users proportionally to the monetary value they have in the system, inspired by proof-of-stake approaches, suggested as an alternative or enhancement to proof-of-work [ 3 , 10 ].
 There is akey difference, however, between Algorand using monetaryvalue as weights and many proof-of-stake cryptocurrencies.
In many proof-of-stake cryptocurrencies, a malicious leader(who assembles a new block)can create a forkin the network,but if caught (e.
g.
, since two versions of the new block aresigned with his key), the leader loses his money.
 The weightsin Algorand, however, are only to ensure that the attackercannot amplify his power by using pseudonyms; as long asthe attacker controls less than 1/3 of the monetary value,Algorand can guarantee that the probability for forks is negligible.
 Algorand may be extended to “detect and punish”```malicious users, but this is not required to prevent forks ordouble spending.
Proof-of-stake avoids the computational overhead ofproof-of-work and therefore allows reducing transaction confirmation time.
 However, realizing proof-of-stake in practiceis challenging [ 4 ].
 Since no work is involved in generatingblocks, a malicious leader can announce one block, and thenpresent some other block to isolated users.
 Attackers mayalso split their credits among several “users”, who mightget selected as leaders, to minimize the penalty when a badleader is caught.
 Therefore some proof-of-stake cryptocurrencies require a master key to periodically sign the correctbranch of the ledger in order to mitigate forks [ 31 ].
 Thisraises significant trust concerns regarding the currency, andhas also caused accidental forks in the past [ 43 ].
 Algorandanswers this challenge by avoiding forks, even if the leaderturns out to be malicious.
Ouroboros [ 30 ] is a recent proposal for realizing proof-ofstake.
 For security, Ouroboros assumes that honest users cancommunicate within some bounded delay (i.
e.
, a stronglysynchronous network).
 Furthermore, it selects some usersto participate in a joint-coin-flipping protocol and assumesthat most of them are incorruptible by the adversary fora significant epoch (such as a day).
 In contrast Algorandassumes that the adversary may temporarily fully control thenetwork and immediately corrupt users in targeted attacks.
``````Trees and DAGs instead of chains.
GHOST [ 47 ], SPECTRE [ 48 ], and Meshcash [ 5 ] are recent proposals for increasing Bitcoin’s throughput by replacing the underlying chainstructured ledger with a tree or directed acyclic graph (DAG)structures, and resolving conflicts in the forks of these datastructures.
 These protocols rely on the Nakamoto consensususing proof-of-work.
 By carefully designing the selectionrule between branches of the trees/DAGs, they are able tosubstantially increase the throughput.
 In contrast, Algorandis focused on eliminating forks; in future work, it may beinteresting to explore whether tree or DAG structures cansimilarly increase Algorand’s throughput.

