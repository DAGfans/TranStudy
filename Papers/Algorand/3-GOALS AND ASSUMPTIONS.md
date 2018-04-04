
## 3 GOALS AND ASSUMPTIONS
## 3 目标和假设 

Algorand allows users to agree on an ordered log of transactions, and achieves two goals with respect to the log:
Algorand允许用户就交易的有序日志达成一致，并在日志方面实现两个目标：

**Safety goal.** With overwhelming probability, all users agree on the same transactions.
 More precisely, if one honest user accepts transaction A (i.e., it appears in the log), then any future transactions accepted by other honest users will appear in a log that already contains A.
 This holds even for isolated users that are disconnected from the network—e.g., by Eclipse attacks [28].
 
**安全目标** 极大的概率下，所有用户会对同样的交易达成一致。
更具体地说，如果一个诚实的用户接受交易A（即它出现在日志中），那么其他诚实用户接受的任何未来交易都将出现在已经包含A的日志中。
即使对于被迫与网络断开连接的孤立用户，例如被Eclipse攻击[28]，也是如此。


**Liveness goal.** In addition to safety, Algorand also makes progress (i.e., allows new transactions to be added to the log) under additional assumptions about network reachability that we describe below.
 Algorand aims to reach consensus on a new set of transactions within roughly one minute.
**活性目标** 除了安全性外，Algorand还在下面描述的有关网络可达性的其他假设下取得进展（即允许将新事务添加到日志中）。
Algorand的目标是在大约一分钟内就新的交易达成共识。


**Assumptions.** Algorand makes standard cryptographic assumptions such as public-key signatures and hash functions.
 Algorand assumes that honest users run bug-free software.
 As mentioned earlier, Algorand assumes that the fraction of money held by honest users is above some threshold h (a constant greater than 2/3), but that an adversary can participate in Algorand and own some money.
 We believe that this assumption is reasonable, since it means that in order to successfully attack Algorand, the attacker must invest substantial financial resources in it.
 Algorand assumes that an adversary can corrupt targeted users, but that an adversary cannot corrupt a large number of users that hold a significant fraction of the money (i.e., the amount of money held by honest, non-compromised users must remain over the threshold).
 
**假设** Algorand基于标准的密码学的假设，如公钥签名和散列函数。
Algorand也假设诚实的用户运行着无错软件。
如前所述，Algorand假设诚实用户持有的资金占比高于某个阈值h（大于2/3的常数），但攻击者可以参与Algorand并拥有一些资金。
我们认为这种假设是合理的，因为这意味着为了成功攻击Algorand，攻击者必须投入大量财力。
Algorand假设攻击者可以破坏目标用户，但是对手不能破坏持有相当大部分资金的大量用户（即，诚实的，未受到损害的用户持有的资金量必须超过阈值）。


To achieve liveness, Algorand makes a “strong synchrony” assumption that most honest users (e.g., 95%) can send messages that will be received by most other honest users (e.g., 95%) within a known time bound.
 This assumption allows the adversary to control the network of a few honest users, but does not allow the adversary to manipulate the network at a large scale, and does not allow network partitions.
 
为了实现活性，Algorand提出了一个“强同步”假设，即大多数诚实用户（例如95％）可以在已知的时间范围内发送消息给大多数其他诚实用户（例如95％）。
这种假设允许攻击者控制一些诚实用户的网络，但不允许攻击者大规模操纵网络，并且不允许网络分区。


Algorand achieves safety with a “weak synchrony” assumption: the network can be asynchronous (i.e., entirely controlled by the adversary) for a long but bounded period of time (e.g., at most 1 day or 1 week).
 After an asynchrony period, the network must be strongly synchronous for a reasonably long period again (e.g., a few hours or a day) for Algorand to ensure safety.
 More formally, the weak synchrony assumption is that in every period of length b (think of b as a day or a week), there must be a strongly synchronous period of length s < b (an s of a few hours suffices).
 
Algorand 可以在“弱同步”假设下实现安全性：网络可以在很长但有限的时间段内（例如最多1天或1周）是异步的（即完全由对手控制）。
在异步时期之后，网络必须在足够长的一段时间内（例如几个小时或一天）保持强同步，以确保Algorand的安全。


Finally, Algorand assumes loosely synchronized clocks across all users (e.g., using NTP) in order to recover liveness after weak synchrony.
 Specifically, the clocks must be close enough in order for most honest users to kick off the recovery protocol at approximately the same time.
 If the clocks are out of sync, the recovery protocol does not succeed.
 
最后，Algorand假定在所有用户之间的时钟是宽松同步的（例如，使用NTP）以便在弱同步之后恢复活性。
具体来说，时钟必须足够接近，以便大多数诚实用户几乎同时启动恢复协议。
如果时钟不同步，则恢复协议不会成功。
