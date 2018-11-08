## 1 Introduction
## 1 介绍

Achieving agreement among a set of distributed hosts lies at the core of countless applications, ranging from Internet-scale services that serve billions of people [12, 30] to cryptocurrencies worth billions of dollars [1].  
在一组分布式主机之间达成协议是无数应用程序的核心，从为数十亿人[12,30]提供服务的互联网模式服务到价值数十亿美元的加密货币[1]。   
To date, there have been two main families of solutions to this problem.     
到目前为止，已经有两种主要的解决方案来解决这个问题。  
Traditional consensus protocols rely on all-to-all communication to ensure that all correct nodes reach the same decisions with absolute certainty.     
传统的共识协议依赖于所有节点间通信，以确保所有正确的节点可以绝对确定地实现相同的决策。  
Because they require quadratic communication overhead and accurate knowledge of membership, they have been difficult to scale to large numbers of participants.  
因为它们需要二次通信开销和准确的成员资格认知，所以参与者很难大量扩展。

On the other hand, Nakamoto consensus protocols [9, 24, 25, 34, 40–43, 49–51] have become popular with the rise of Bitcoin.  
另一方面，Nakamoto共识协议[9,24,25,34,40-43,49-51]随着比特币的兴起而变得流行。     
These protocols provide a probabilistic safety guarantee:   
这些协议提供了概率安全保证：  
Nakamoto consensus decisions may revert with some probability $\epsilon $.   
Nakamoto共识决策可能以某个概率$\epsilon $恢复。
  
A protocol parameter allows this probability to be rendered arbitrarily small, enabling high-value financial systems to be constructed on this foundation.   
协议参数允许这种可能性被表示为任意小，而在这个基础上构建高价值的金融系统。  
This family is a natural fit for open, permissionless settings where any node can join the system at any time.   
该系列非常适合开放，无权限设置，任何节点都可以随时加入系统。    
Yet, these protocols are quite costly, wasteful, and limited in performance.    
然而，这些协议非常昂贵，浪费并且性能有限。   
By construction, these protocols cannot quiesce:   
通过构建，这些协议无法停止：  
their security relies on constant participation by miners, even when there are no decisions to be made.    
即使没有做出决定，他们的安全也依赖于矿工的不断参与。   
Bitcoin currently consumes around 63.49 TWh/year [22], about twice as all of Denmark [15].   
比特币目前消耗约63.49TWh/年[22]，约为丹麦每年消耗的两倍[15]。    
Moreover, these protocols suffer from an inherent scalability bottleneck that is difficult to overcome through simple reparameterization [18].  
此外，这些协议存在固有的可扩展性瓶颈，难以通过简单的重设参数来克服[18]。

This paper introduces a new family of consensus protocols.   
本文介绍了一个新的共识协议系列。  
Inspired by gossip algorithms, this new family gains its safety through a deliberately metastable mechanism.  
受到gossip算法的启发，这个新系列的协议通过使自己故意处于亚稳态机制而获得了安全性。  
Specifically, the system operates by repeatedly sampling the network at random, and steering the correct nodes towards the same outcome.   
具体而言，系统通过在网络上重复采样来进行随机操作，并指导正确的节点朝向相同的结果。  
Analysis shows that metastability is a powerful, albeit non-universal,  technique:   
分析表明，尽管没有普及，亚稳态机制仍是一种强大的技术。  
it can move a large network to an irreversible state quickly,  though it is not always guaranteed to do so.  
它可以将大型网络快速移动到不可逆转的状态，但并不能一直保证可以这样做。

Similar to Nakamoto consensus, this new protocol family provides a probabilistic safety guarantee, using a tunable security parameter that can render the possibility of a consensus failure arbitrarily small.  
与Nakamoto共识类似，这个新协议系列提供了一定概率的安全保证，使用可调整的安全参数，可以使共识失败的可能性变成任意小。  
Unlike Nakamoto consensus, the protocols are green, quiescent and efficient;   
与Nakamoto共识不同，该协议是绿色，静止和高效的;  
they do not rely on proof-of-work [23] and do not consume energy when there are no decisions to be made.   
他们不依赖于工作量证明[23]，并且在没有做出决定时不消耗能量。  
The efficiency of the protocols stems from two sources:   
协议的效率源于两个来源：  
they require communication overheads ranging from $O(knlogn)$ to $O(kn)$ for some small security parameter $k$, and they establish only a partial order among dependent transactions.  
它们需要通信开销的范围从$O(knlogn)$到$O(kn)$对于某个小的安全参数$k$来说，并且它们在独立的交易之间只建立一个偏序。

This combination of the best features of traditional and Nakamoto consensus involves one significant tradeoff: 
liveness for conflicting transactions.   
传统和Nakamoto共识最佳特征的这种组合涉及一个重要的权衡：冲突交易的活跃性。  
Specifically, the new family guarantees liveness only for virtuous transactions, i.e. those issued by correct clients and thus guaranteed not to conflict with other transactions.   
具体而言，新协议系列仅保证良性交易的活性，即由正确的客户发布的交易，从而保证不与其他交易发生冲突。  
In a cryptocurrency setting, cryptographic signatures enforce that only a key owner is able to create a transaction that spends a particular coin.   
在加密货币设置中，加密签名强制只有密钥所有者能够创建花费特定货币的交易。  
Since correct clients follow the protocol as prescribed, they are guaranteed both safety and liveness.   
由于正确的用户遵循规定的协议，他们的安全和活性均被保证。  
In contrast, the protocols do not guarantee liveness for rogue transactions,  submitted by Byzantine clients, which conflict with one another.   
相比之下，协议并不保证拜占庭用户提交的流氓交易的活性，这些交易彼此冲突。  
Such decisions may stall in the network, but have no safety impact on virtuous transactions.   
这样的决定可能使流氓交易在网络中停滞不前，但对良性交易没有安全影响。  
We show that this is a sensible tradeoff, and that resulting system is sufficient for building complex payment systems.  
我们证明这是一个合理的权衡，并且由此产生的系统足以构建复杂的支付系统。

Overall, this paper makes one significant contribu- tion:   
总的来说，本文做出了重要贡献：  
a brand new family of consensus protocols suitable for cryptocurrencies, based on randomized sampling and metastable decision.
基于随机抽样和亚稳态决策的适用于加密货币的全新共识协议系列。  
The protocols provide a strong probabilistic safety guarantee, and a guarantee of liveness for correct clients.  
这些协议提供了强大的概率安全保证，并保障了正确客户的活性。

The next section provides intuition behind the new protocols, Section 3 provides proofs for safety and liveness, Section 4 describes a Bitcoin-like payment system, Section 5 evaluates the system, Section 6 presents related work, and finally, Section 7 summarizes our contributions.  
下一节提供了新协议背后的思路，第3节提供了安全性和活性的证明，第4节描述了类比特币支付系统，第5节评估了系统，第6节介绍了相关工作，最后，第7节总结了我们的贡献。
