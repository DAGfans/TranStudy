9

IMPLEMENTATION

We implemented a prototype of Algorand in C++, consist- ing of approximately 5,000 lines of code. We use the Boost ASIO library for networking. Signatures and VRFs are im- plemented over Curve 25519 [6], and we use SHA-256 for a hash function. We use the VRF outlined in Goldberg et al [27: §4].

In our implementation each user connects to 4 random peers, accepts incoming connections from other peers, and gossips messages to all of them. This gives us 8 peers on average. We currently provide each user with an “address book” file listing the IP address and port number for every user’s public key. In a real-world deployment we imagine users could gossip this information, signed by their keys, or distribute it via a public bulletin board. This naïve design of the gossip protocol in our prototype implementation is po- tentially susceptible to Sybil attacks, since it does not prevent an adversary from joining the gossip network with a large number of identities. We leave the problem of implementing a Sybil-resistant gossip network to future work.

One difference between our implementation and the pseu- docode shown in §7 lies in the BinaryBA⋆() function. The pseudocode in Algorithm 8 votes in the next 3 steps after reaching consensus. For efficiency, our implementation in- stead looks back to the previous 3 steps before possibly re- turning consensus in a future step. This logic produces equiv- alent results but is more difficult to express in pseudocode.

Figure 4 shows the parameters in our prototype of Algo- rand; we experimentally validate the timeout parameters in §10. h = 80% means that an adversary would need to control 20% of Algorand’s currency in order to create a fork. By analogy, in the US, the top 0.1% of people own about 20% of the wealth [40], so the richest 300,000 people would have to collude to create a fork.

λ priority should be large enough to allow block proposers to gossip their priorities and proofs. Measurements of mes- sage propagation in Bitcoin’s network [18] suggest that gossiping 1 KB to 90% of the Bitcoin peer-to-peer network takes about 1 second. We conservatively set λ priority to 5 seconds.

λ block ensures that Algorand can make progress even if the block proposer does not send the block. Our experiments (§10) show that about 10 seconds suffices to gossip a 1 MB block. We conservatively set λ block to be a minute.

λ step should be high enough to allow users to receive messages from committee members, but low enough to allow Algorand to make progress (move to the next step) if it does not hear from sufficiently many committee members. We conservatively set λ step to 20 seconds. We set λ stepvar , the estimated variance in BA⋆ completion times, to 10 seconds.
