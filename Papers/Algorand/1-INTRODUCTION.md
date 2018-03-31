* Source: https://people.csail.mit.edu/nickolai/papers/gilad-algorand-eprint.pdf
* TranStudy: https://github.com/DAGfans/TranStudy/edit/master/Papers/Algorand/1-INTRODUCTION.md




## 1 INTRODUCTION

## 1 介绍

Cryptographic currencies such as Bitcoin can enable new applications, such as smart contracts [ 24 , 49 ] and fair protocols [ 2 ], can simplify currency conversions [ 12 ], and can avoid trusted centralized authorities that regulate transactions. 
However, current proposals suffer from a trade-off between latency and confidence in a transaction. 
For example, achieving a high confidence that a transaction has been confirmed in Bitcoin requires about an hour long wait [ 7 ].
On the other hand, applications that require low latency cannot be certain that their transaction will be confirmed, and must trust the payer to not double-spend [45].

像比特币这样的加密货币可以产生新的应用，如智能合约[24,49]和公平协议[2]，能够简化货币转换[12]，并且可以不需要可信中央机构来管理交易。
然而，目前的方案都需要在延迟和信任之间的权衡。 
例如，在比特币中以较高可信度确认交易需要大约一个小时的等待时间[7]。
另一方面，低延迟的应用无法确定他们的交易是否得到确认，并且必须相信支付方不会双花[45]。

Double-spending is the core problem faced by any cryptocurrency, where an adversary holding $1 gives his $1 to two different users. 
Cryptocurrencies prevent double-spending by reaching consensus on an ordered log (“blockchain”) of transactions. 
Reaching consensus is difficult because of the open setting: since anyone can participate, an adversary can create an arbitrary number of pseudonyms (“Sybils”) [ 21 ], making it infeasible to rely on traditional consensus protocols [15] that require a fraction of honest users.

双花是任何加密货币所面临的核心问题，即持有1美元的攻击者将1美元支付两个不同的用户。
加密货币通过在有序的交易日志（“区块链”）上达成共识来防止双花。
在公开的环境下达成共识是困难的：因为任何人都可以参与，攻击者可以创建任意数量的虚假账号（“Sybils”）[21]，导致依靠传统的需要有部分诚实用户的共识协议[15]是不可行的。

Bitcoin [ 41 ] and other cryptocurrencies [ 23 , 53 ] address this problem using proof-of-work (PoW), where users must repeatedly compute hashes to grow the blockchain, and the longest chain is considered authoritative. 
PoW ensures that an adversary does not gain any advantage by creating pseudonyms. 
However, PoW allows the possibility of forks, where two different blockchains have the same length, and neither one supersedes the other. 
Mitigating forks requires two unfortunate sacrifices: the time to grow the chain by one block must be reasonably high (e.g., 10 minutes in Bitcoin), and applications must wait for several blocks in order to ensure their transaction remains on the authoritative chain (6 blocks are recommended in Bitcoin [ 7 ]). 
The result is that it takes about an hour to confirm a transaction in Bitcoin.

比特币[41]和其他加密货币[23,53]使用工作量证明（PoW）解决了这个问题，用户必须重复计算散列来延伸区块链，最长的链被认为是权威的。
PoW确保攻击者不会通过创建虚假账号获得任何优势。
但是，PoW允许分叉，有可能其中两个不同的区块链具有相同的长度，并且两个区块链都不相互替代。
为了缓解分叉有两个沉重的代价：在链上延伸一个时间必须相当长（例如比特币为10分钟），并且应用程序必须等待几个区块才能确保其交易保存在权威的链上（在比特币[7]中推荐6个区块）。
导致确认比特币交易大约需要一个小时。

This paper presents Algorand, a new cryptocurrency designed to confirm transactions on the order of one minute.
The core of Algorand uses a Byzantine agreement protocol called BA⋆that scales to many users, which allows Algorand to reach consensus on a new block with low latency and without the possibility of forks. 
A key technique that makes BA⋆ suitable for Algorand is the use of verifiable random functions (VRFs) [ 38 ] to randomly select users in a private and non-interactive way.
BA⋆was previously presented at a workshop at a high level [ 37 ], and a technical report by Chen and Micali [16] described an earlier version of Algorand.

本文介绍了Algorand，一种新的加密货币，旨在一分钟的规模内的确认交易。
Algorand的核心在于使用了称为BA* 的拜占庭协定的协议，它可以扩展到许多用户，这使得Algorand能够以低延迟达成共识，而且不会产生分叉。
使BA* 适用Algorand的一种关键技术是用可验证的随机函数（VRF）[38]来以私有和非交互的方式随机选择用户.
BA* 之前在高级别的研讨会上提出[37 ], 并且Chen和Micali [16]的技术报告描述了Algorand的早期版本。

Algorand faces three challenges. 
First, Algorand must avoid Sybil attacks, where an adversary creates many pseudonyms to influence the Byzantine agreement protocol.
Second,BA⋆must scale to millions of users, which is far higher than the scale at which state-of-the-art Byzantine agreement protocols operate. 
Finally, Algorand must be resilient to denial-of-service attacks, and continue to operate even if an adversary disconnects some of the users [29, 51].

Algorand面临三大挑战。
首先，Algorand必须避免女巫攻击，在这种攻击中，攻击者会创建许多假账号来影响拜占庭协议。
其次，BA* 必须扩展到数百万用户，这远远高于最先进的拜占庭协议的运行规模。
最后，Algorand必须能够抵御拒绝服务攻击，并且即使攻击者切断某些用户的连接也可以继续运行[29,51]。

Algorand addresses these challenges using several techniques, as follows.

Algorand使用以下几种技术应对这些挑战。

**Weighted users** 
To prevent Sybil attacks, Algorand as signs a weight to each user.
BA⋆is designed to guarantee consensus as long as a weighted fraction (a constant greater than 2/3) of the users are honest. 
In Algorand, we weigh users based on the money in their account. 
Thus, as long as more than some fraction (over 2/3) of the money is owned by honest users, Algorand can avoid forks and double-spending.

**有权重的用户** 
为了防止女巫，Algorand对每个用户都加以权衡.
BA* 被设计为只要诚实用户的总权重达到一定比例（某大于2/3的常数），就能保证一致性。
在Algorand，我们根据用户账户中的资金来衡量用户的权重。
因此，只要有诚实用户拥有一部分（超过2/3）的钱，Algorand就可以避免分叉和双花。

**Consensus by committee** 
BA⋆achieves scalability by choosing a committee—a small set of representatives randomly selected from the total set of users—to run each step of its protocol. 
All other users observe the protocol messages, which allows them to learn the agreed-upon block.
BA⋆chooses committee members randomly among all users based on the users’ weights. 
This allows Algorand to ensure that a sufficient fraction of committee members are honest.
However, relying on a committee creates the possibility of targeted attacks against the chosen committee members.

**委员会的共识** 
BA* 通过选择一个委员会（一组从全体用户中随机选出的一些代表）来执行协议的每一步，来提高可扩展性。
所有其他用户观察协议消息，这使他们能够得知已商定的块。
BA* 根据用户的权重在所有用户中随机选择委员会成员。
因此Algorand可以确保有足够比例的委员会成员是诚实的。
但是，依靠委员会可能会让选定的委员会成员遭受有针对性的攻击。

**Cryptographic sortition** 
To prevent an adversary from targeting committee members,BA⋆selects committee members in a private and non-interactive way. 
This means that every user in the system can independently determine if they are chosen to be on the committee, by computing a function (a VRF [ 38 ]) of their private key and public information from the blockchain. 
If the function indicates that the user is chosen, it returns a short string that proves this user’s committee membership to other users, which the user can include in his network messages. 
Since membership selection is non-interactive, an adversary does not know which user to target until that user starts participating in BA⋆.

**加密的抽签**
为了防止敌人把目标锁定在委员会成员身上，BA以私有和非互动的方式选择委员会成员。
这意味着系统中的每个用户都可以通过计算他们私钥的函数（VRF [38]）和区块链中的公共信息来独立确定他们是否被选中参加委员会。
如果该函数指示用户被选中，它将返回一个短字符串，该字符串向其他用户证明此用户的委员会成员资格，用户可以在其网络消息中包含该字符串。
由于会员选择是非交互式的，因此在用户开始参与BA*之前，攻击者不知道要定位哪个用户。

**Participant replacement.** 
Finally, an adversary may target a committee member once that member sends a message in BA⋆.
BA⋆mitigates this attack by requiring committee members to speak just once. 
Thus, once a committee member sends his message (exposing his identity to an adversary), the committee member becomes irrelevant to BA⋆. 
BA⋆ achieves this property by avoiding any private state (except for the user’s private key), which makes all users equally capable of participating, and by electing new committee members for each step of the Byzantine agreement protocol.

**参与者的更换**
最后，一旦某委员会成员在BA*发送信息，攻击者可能会定位该名成员。
BA* 通过要求委员会成员只发一次消息来消除这种攻击。
因此，一旦委员会成员发送他的消息（将他的身份暴露给对手），它就不参与BA* 了。
BA* 通过避免任何私有状态（用户的私钥除外），使所有用户都有能力参与，并通过为拜占庭协议的每一步选举新的委员会成员来实现此功能。

We implement a prototype of Algorand and BA⋆, and use it to empirically evaluate Algorand’s performance. Experimental results running on 1,000 Amazon EC2 VMs demonstrate that Algorand can confirm a 1 MByte block of transactions in∼22 seconds with 50,000 users, that Algorand’s latency remains nearly constant when scaling to half a million users, that Algorand achieves 125×the transaction throughput of Bitcoin, and that Algorand achieves acceptable latency even in the presence of actively malicious users.

我们实现了Algorand和BA* 的原型，并用它来评估Algorand的真实表现。 
在1,000台亚马逊EC2虚拟机上运行的实验结果表明，在Algorand下，50,000个用户可以在约22秒内确认一个1M字节的交易块，当扩展到50万用户时，Algorand的延迟几乎保持不变，Algorand实现了比特币125倍的交易吞吐量 ，Algorand甚至在存在主动恶意用户的情况下也能达到可接受的延迟。
