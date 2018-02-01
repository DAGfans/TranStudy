[Source](https://steemkr.com/smart/@alexbafana/smart-contract-languages-comparison "Permalink to Smart-Contract Languages Comparison — Steemkr")
Author: https://steemkr.com/@alexbafana

# Smart-Contract Languages Comparison

![][1]

The traditional understanding of a contract is a written or spoken agreement enforceable by law. In most business cases, contracts are documents [1] that identify the contracting parties uniquely, a service that is offered for some form of compensation that is usually monetary, and a set of additional clauses such as service-delivery dates, penalties for delivery failure, compensation clauses, and so on. Subsequent transactions are trust-based and contracting parties usually consider contracts as a symbol for an existing business deal. Another problem with the traditional form of setting up and managing contracts is that they are often underspecified. Most importantly, traditional contracts do not provide sufficient details about the actual transaction process and consequently, frictions  with conflicts between the contracting parties are very likely.   

### Smart Contracts and Decentralized Autonomous Organizations

A solution for the listed issues pertaining to managing traditional contracts is to aim for an automation in the form of smart electronic contracts [2] that govern business transactions between decentralized autonomous organizations (DAO) [3]. A smart contract is a computerized transaction protocol [4] to execute contract terms. To achieve non-repudiation and fact-tracking of a consensual smart-contract agreement, blockchain technology [5] is suitable. Note that a smart contract is a socio-technical artifact and can not only be considered from a strictly technical perspective. Still, pertaining to the latter, the blockchain is a distributed database for independently verifying the chain of ownership of artifacts [6] in hash values that result from cryptographic digests. 

Recently, an experiment with a DAO has been developed with the smart contract technology of [Ethereum][2]. This DAO served as a [crowd-funding project][3] and was hacked because of security flaws, resulting in a loss of ca. $50 million. This incident shows it is not enough to merely equip the protocol layer on top of a blockchain with a Turing-complete language such as [Solidity][4] to realize smart-contract management. Instead, it is important to study if these languages are suitable and expressive enough [7] for smart-contract formalization. Suitability means that a languages comprise ontological concepts and properties to allow the formulation of real-world contracts in many perspectives in a machine-readable way. Expressiveness means the constructs of a language have formal mathematical clarity for ensuring uniform enactment behavior by different enactment engines. The latter is also important for enabling tool-supported verifications for preventing a repeated DAO scandal. 

### Smart-contract ontology

In a recent publication [7], a top-down developed smart-contract ontology serves to evaluate an academic and experimental language termed eSourcing Markup Language (eSML). The core ontological concepts answer the conceptual questions of Who, Where and What. The Who-concept legally clearly identifies the contracting parties by including the class _party_. Parties are actors that have rights and obligations that are listed in the eSourcing configuration. Concerning the relationship cardinalities, it is defined that at least two party specifications must be part of a  contract. It is also possible to have more than two parties defined. For the Where-concept, we distinguish two basic aspects of the electronic contracting context, i.e., the business context and the legal context. In addition, a third subsection optionally references other electronic contracting provisions that are not part of the core legal and business context. Finally, the What-concept contains concepts related to the exchanged values and transaction conditions. Two main subsections of the What-concept are the exchanged value and the corresponding exchange provisions for the value exchange. These classes are defined separately for every respective contractual party involved in contracting. 

### Smart-Contract Languages

With respect to smart-contract languages, we consider the [Solidity][5] language of Ethereum and [Eris][6]. The latter is a fork of Ethereum with the objective to turn the public blockchain of Ethereum into a permissioned version to be more attractive for reclusive application domains such as in the banking industry. This forking explains why Eris also considers Solidity for smart-contract specifications.   

More recently, Synereo [8] emerged from a failed Ethereum-based implementation of a censorship-free social media such as Facebook. The failure reasons are lack of Ethereum scalability because of full replication of the blockchain into every node, combined with costly proof-of-work block validations. Synereo uses [RChain][7] that is a concurrent and compositional blockchain. Furthermore, Synereo adopts as a smart-contract language Rholang [9] that is a concurrency-oriented programming language with a focus on message-passing and asynchrony. 

Another smart-contract system is [Lisk][8] that differs from Ethereum in that every blockchain application is on its own sidechain. Informally, sidechaining allows tokens from one blockchain to be securely used within a completely separate blockchain and still moved back to the original chain. Both Lisk and Solidity have in common that both have a syntax similar to JavaScript for smart-contract specifications.   For the ontology-based [7] language comparison, we do not consider Lisk and it suffices to state that the latter is a Turing-complete option and in one cumbersome way or another, it is possible to formulate smart contracts. Solidity is currently the most mature solution in use for industry-projects. Rholang exists for now as a syntactic language specification [9] and is backed by profound scholarly publications [11,12,13]. Rholang must still be realized as an applicable language. Regardless, based on the documentation [7], we are able to focus on a comparison of suitability and expressiveness between Solidity and Rholang against the existing smart-contract ontology.

### Rholang suitability and expressiveness

The syntactic specification [7] of Rholang suggests there is a considerable intersection with the ontological concepts and properties for smart-contract languages. Rholang considers patterns for the storage and retrieval of information via channels. The latter also holds in the eSML evaluation that too adopts patterns for channels and data-flow. Differently to Rholang, eSML also adopts control-flow patterns. Just as eSML, Rholang assumes that contracts must be process aware and adopts  again patterns for that. In eSML, matched processes are public views [10] that are subsets of larger in-house processes in the domains of respective collaborating parties. Unfortunately, the Rholang documentation does not clarify what matching of processes concretely means. For example, the matching of processes from collaborating parties needs to establish what contained activities are semantically equivalent. Activities also have lifecycles that may differ, e.g., in one domain an activity can be enabled, chosen, enacted, halted, resumed, terminated completed while only a subset of such lifecycle steps are assumed for activities in the domain of another matched process. 

Rholang falls short in not adopting any syntax provisions for integrating business rules and policies. However, rules and policies should be an important part for smart contracts that are equivalent to the formulation of obligations in traditional contracts that are not machine readable. To give the benefit of doubt, we may infer that rules and policies could be part of an extended Rholang definition that details the properties and concepts of processes. 

 Notable is also that a smart contract must uniquely identify who the collaborating parties are and the currently specified Rholang syntax does not appear to provide any concepts and properties for that. The unique identification of collaborating parties should also incorporate the ability to specify what the required competencies, authorizations, and so on, of roles are that carry out activities. All of that is part of the so-called resource perspective of a process for which Rholang also lacks properties and concepts in the currently existing syntax definition. 

### Solidity suitability and expressiveness

Because of the Turing-complete nature of Solidity, it is in principle possible to achieve in some more or less cumbersome way a support or all concepts and properties of the smart-contract ontology that eSML embodies. However, concepts that also Rholang recognizes, such as pattern-based design, process awareness, matching of processes, etc., are not adopted in any way in Solidity. With respect to inventing cumbersome workarounds, a recent conference-paper publication [14] uses Solidity to demonstrate the feasibility of untrusted business-process monitoring and execution in smart contracts. 

 One must stress that Solidity has historically not been backed by formal verification means such as the scholarly papers of Rholang [11,12,13]. Without such expressiveness, it is not possible to know ahead of enactment if a contract is correct and free of security issues, which explains why the DAO scandal happened. The latter has triggered only very recently the development and application of dedicated tools such as [Why3][9], [Solidifier][10], or [Casper][11] that likely leads to a shift from proof-of-work towards proof-of-stake for Ethereum all together. 

It is a general observation that Solidity is a language with a focus on mostly low-level blockchain-manipulation commands with JavaScript-style syntax. Still, it is possible to import third-party APIs and perform external function calls. So-called external functions in Solidity are part of a smart-contract interface that can be called from other contracts and via transactions. Thus, Solidity and Ethereum function as a "spider in the web" and thereby allow for orchestrating a larger information infrastructure to compensate in combination for many inherent suitability shortcomings of Solidity in a comparison against the smart-contract ontology that eSML embodies. 

### Conclusion

This article compares the suitability and expressiveness of smart-contract languages with a focus on Rholang and Solidity. The comparison is based on a scholarly published smart-contract ontology that a purely academic and experimental eSML language embodies. In summary, although Rholang only exists for now as a syntactic definition that formal scholarly papers back, one can expect a high degree of suitability and expressiveness once a machine-readable language will be released. Solidity is a mature smart contract language with JavaScript-related syntax that industry already feverishly adopts. However, while Solidity is Turing complete, the suitability comparison against the smart-contract ontology that eSML embodies, is not as convincing as for Rholang. The historic lack of formal-verification considerations in Solidity also reveals expressiveness deficiencies that allow for major security breaches in smart contracts that have already occurred in real life. 

The inherent suitability and expressiveness of smart-contract languages also has implications on the underlying enactment machinery. Studying the documentation of [Synereo][12] reveals the not yet  implement architecture stack promises to be superior in comparison to Ethereum. The latter has performance issues and does not scale in the current form. Synereo theoretically solves performance and scalability issues with scalable RChains, suitable and expressive Rholang, and so on, If the vision of the Synereo architecture stack is translated successfully into an operational system then we can expect a respectable leap in developing a totally new generation of trustless and distributed Internet applications. These next-generation applications will hopefully also realize the desirable vision of censorship-immune social media that is currently used by coercive organizations as information-warfare systems to achieve horrific and odious societal objectives. Consequently, it is good to observe that considerable energy is channeled into the discovery of innovations that tackle and reverse such societal problems.  

**References** 

[1] T. Roxenhall and P. Ghauri. Use of the written contract in long-lasting business relationships. Industrial Marketing Management, 33(3):261 – 268, 2004. 

[2] N. Szabo. Formalizing and securing relationships on public networks. First Monday, 2(9), 1997. 

[3] V. Butterin. A next-generation smart contract and decentralized application platform, 2014. 

[4] M. Swan. Blockchain thinking: The brain as a DAO (Decentralized Autonomous Organization). In Texas Bitcoin Conference, pages 27–29, 2015. 

[5] S. Nakamoto. Bitcoin: A peer-to-peer electronic cash system. Consulted, 1(2012):28, 2008. 

[6] B.S. Panikkar, S. Nair, P. Brody, and V. Pureswaran. Adept: An IoT practitioner perspective, 2014. 

[7] A. Norta, L. Ma, Y. Duan, A. Rull, M. Kolvart, and K. Taveter. Econtractual choreography-language properties towards cross-organizational business collaboration. Journal of Internet Services and Applications, 6(1):1–23, 2015. 

[8] D. Konforty, Y. Adam, D. Estrada, and L. G. Meredith. Synereo: The decentralized and distributed social network. 2015. 

[9] L.G. Meredith, J. Pettersson, G. Stephenson, Contracts, Composition, and Scaling, The Rholang 0.1 specification. 2016   

[10] R. Eshuis, A. Norta, and R. Roulaux. "Evolving process views." _Information and Software Technology_ 80 (2016): 20-35. 

[11] L.G. Meredith and M. Radestock. "A reflective higher-order calculus." _Electronic Notes in Theoretical Computer Science_ 141.5 (2005): 49-67. 

[12] L.G. Meredith and M. Radestock. "Namespace logic: A logic for a reflective higher-order calculus." _International Symposium on Trustworthy Global Computing_. Springer Berlin Heidelberg, 2005. 

[13] M. Stay, and L.G. Meredith. "Logic as a distributive law." _arXiv preprint arXiv:1610.02247_ (2016). 

[14] I. Weber, X. Xu, R. Riveret, G. Governatori, A. Ponomarev, and J. Mendling, (2016, September). Untrusted business process monitoring and execution using blockchain. In _International Conference on Business Process Management_ (pp. 329-347). Springer International Publishing. 

[1]: https://steemitimages.com/0x0/http://group-global.org/sites/default/files/publications/trust_shutterstock.jpg
[2]: https://ethereum.org/
[3]: https://www.wired.com/2016/06/50-million-hack-just-showed-dao-human/
[4]: https://solidity.readthedocs.io/en/develop/
[5]: http://solidity.readthedocs.io/en/latest/
[6]: https://monax.io/
[7]: https://blog.synereo.com/2016/09/05/meet-rchain-the-first-scalable-blazing-fast-turing-complete-blockchain/
[8]: https://lisk.io/
[9]: http://why3.lri.fr/
[10]: https://hack.ether.camp/idea/solidifier---formal-verification-of-solidity-programs
[11]: https://www.reddit.com/r/ethereum/comments/3flj4x/if_ethereum_adopts_casper_proof_of_stake_it_is/
[12]: https://blog.synereo.com/2016/09/29/standing-on-the-shoulder-of-giants-synereos-full-tech-stack-explained/
