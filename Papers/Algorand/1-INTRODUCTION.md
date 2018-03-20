
## 1 INTRODUCTION

Cryptographic currencies such as Bitcoin can enable new applications, such as smart contracts [ 24 , 49 ] and fair protocols [ 2 ], can simplify currency conversions [ 12 ], and can avoid trusted centralized authorities that regulate transactions. 
However, current proposals suffer from a trade-off between latency and confidence in a transaction. For example, achieving a high confidence that a transaction has been
confirmed in Bitcoin requires about an hour long wait [ 7 ].
On the other hand, applications that require low latency cannot be certain that their transaction will be confirmed, and must trust the payer to not double-spend [45].

Double-spending is the core problem faced by any cryptocurrency, where an adversary holding $1 gives his $1 to two different users. 
Cryptocurrencies prevent double-spending by reaching consensus on an ordered log (“blockchain”) of transactions. 
Reaching consensus is difficult because of the open setting: since anyone can participate, an adversary can create an arbitrary number of pseudonyms (“Sybils”) [ 21 ], making it infeasible to rely on traditional consensus protocols [15] that require a fraction of honest users.

Bitcoin [ 41 ] and other cryptocurrencies [ 23 , 53 ] address this problem using proof-of-work (PoW), where users must repeatedly compute hashes to grow the blockchain, and the longest chain is considered authoritative. 
PoW ensures that an adversary does not gain any advantage by creating pseudonyms. 
However, PoW allows the possibility offorks, where two different blockchains have the same length, and neither one supersedes the other. 
Mitigating forks requires two unfortunate sacrifices: the time to grow the chain by one
block must be reasonably high (e.g., 10 minutes in Bitcoin), and applications must wait for several blocks in order to ensure their transaction remains on the authoritative chain
(6 blocks are recommended in Bitcoin [ 7 ]). 
The result is that it takes about an hour to confirm a transaction in Bitcoin.

This paper presents Algorand, a new cryptocurrency designed to confirm transactions on the order of one minute.
The core of Algorand uses a Byzantine agreement protocol calledBA⋆that scales to many users, which allows Algorand to reach consensus on a new block with low latency and without the possibility of forks. A key technique that makes BA⋆suitable for Algorand is the use of verifiable random functions (VRFs) [ 38 ] to randomly select users in a private and non-interactive way.BA⋆was previously presented at a workshop at a high level [ 37 ], and a technical report by Chen and Micali [16] described an earlier version of Algorand.

Algorand faces three challenges. First, Algorand must avoid Sybil attacks, where an adversary creates many pseudonyms to influence the Byzantine agreement protocol.
Second,BA⋆must scale to millions of users, which is far higher than the scale at which state-of-the-art Byzantine agreement protocols operate. 
Finally, Algorand must be resilient to denial-of-service attacks, and continue to operate even if an adversary disconnects some of the users [29, 51].

Algorand addresses these challenges using several techniques, as follows.

**Weighted users.** To prevent Sybil attacks, Algorand as signs a weight to each user.BA⋆is designed to guarantee consensus as long as a weighted fraction (a constant greater than 2/3) of the users are honest. In Algorand, we weigh users based on the money in their account. 
Thus, as long as more than some fraction (over 2/3) of the money is owned by honest users, Algorand can avoid forks and double-spending.

**Consensus by committee.** BA⋆achieves scalability by choosing a committee—a small set of representatives randomly selected from the total set of users—to run each step of its protocol. All other users observe the protocol messages, which allows them to learn the agreed-upon block.
BA⋆chooses committee members randomly among all users based on the users’ weights. 
This allows Algorand to ensure that a sufficient fraction of committee members are honest.
However, relying on a committee creates the possibility of targeted attacks against the chosen committee members.

**Cryptographic sortition.** To prevent an adversary from targeting committee members,BA⋆selects committee members in a private and non-interactive way. 
This means that every user in the system can independently determine if they are chosen to be on the committee, by computing a function (a VRF [ 38 ]) of their private key and public information from the blockchain. If the function indicates that the user is chosen, it returns a short string that proves this user’s committee membership to other users, which the user can
include in his network messages. Since membership selection is non-interactive, an adversary does not know which user to target until that user starts participating inBA⋆.

**Participant replacement.** Finally, an adversary may target a committee member once that member sends a message inBA⋆.BA⋆mitigates this attack by requiring committee
members to speak just once. 
Thus, once a committee member sends his message (exposing his identity to an adversary),
the committee member becomes irrelevant toBA⋆. BA⋆ achieves this property by avoiding any private state (except for the user’s private key), which makes all users equally capable of participating, and by electing new committee members for each step of the Byzantine agreement protocol.

We implement a prototype of Algorand andBA⋆, and use it to empirically evaluate Algorand’s performance. Experimental results running on 1,000 Amazon EC2 VMs demonstrate that Algorand can confirm a 1 MByte block of transactions in∼22 seconds with 50,000 users, that Algorand’s latency remains nearly constant when scaling to half a million users, that Algorand achieves 125×the transaction throughput of Bitcoin, and that Algorand achieves acceptable latency even in the presence of actively malicious users.
