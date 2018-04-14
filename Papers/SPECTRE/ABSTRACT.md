# SPECTRE:

### Serialization of Proof-of-work Events: Conﬁrming Transactions via Recursive Elections
### 工作量证明的序列化: 通过递归投票来确认交易

### Yonatan Sompolinsky, Yoad Lewenberg, and Aviv Zohar

### School of Engineering and Computer Science, The Hebrew University of Jerusalem, Israel
### 以色列耶路撒冷希伯来大学工程与计算机科学学院

### {yoni sompo,yoadlew,avivz}@cs.huji.ac.il

## Abstract
## 摘要

Bitcoin utilizes the Nakamoto Consensus to achieve agreement on a consistent set of transactions, in the permissionless setting, where anyone can participate in the protocol anonymously. 
Since its rise, many other permissionless consensus protocols have been proposed. 
We present SPECTRE, a new protocol for the consensus core of cryptocurrencies that remains secure even under high throughput and fast conﬁrmation times. 
At any throughput, SPECTRE is resilient to attackers with up to 50% of the computational power (reaching the limit deﬁned by network congestion and bandwidth constraints). 
SPECTRE can operate at arbitrarily high block creation rates, which implies that its transactions conﬁrm in mere seconds (limited mostly by the round-trip-time in the network).

比特币利用中本聪共识在无需许可的环境下就一系列交易达成协议，任何人都可以匿名参与协议。 
自从它的崛起以来，已经提出了许多其他无需许可的共识协议。 
我们推出了SPECTRE，一个即使在高吞吐量和快速确认时间下仍保持安全的加密货币的共识核心的新协议。 
在任何吞吐量情况下，SPECTRE对具有高达50％计算能力的攻击者的弹性（达到网络拥塞和带宽限制所定义的极限）。 
SPECTRE可以以任意高的数据块创建速率运行，这意味着其交易仅在几秒钟内（主要受限于网络中的往返时间）确认。

SPECTRE’s underlying model falls into the category of partial synchronous networks: its security depends on the existence of some bound on the delivery time of messages between honest participants, but the protocol itself does not contain any parameter that depends on this bound. 
Hence, while other protocols that do encode such parameters must operate with extreme safety margins, SPECTRE converges according to the actual network delay.

SPECTRE的基础模型属于部分同步网络的范畴：其安全性取决于诚实参与者之间的消息传递时间的范围，但协议本身不包含依赖于该范围的任何参数。 
因此，尽管编码这些参数的其他协议必须以极高的安全系数运行，但SPECTRE会根据实际的网络延迟收敛。

Key to SPECTRE’s achievements is the fact that it satisﬁes weaker properties than classic consensus requires. 
In the conventional paradigm, the order between any two transactions must be decided and agreed upon by all non-corrupt nodes. 
In contrast, SPECTRE only satisﬁes this with respect to transactions performed by honest users. 
We observe that in the context of money, two conﬂicting payments that are published concurrently could only have been created by a dishonest user, hence we can afford to delay the acceptance of such transactions without harming the usability of the system. 
Our framework formalizes this weaker set of requirements for a cryptocurrency’s distributed ledger. 
We then provide a formal proof that SPECTRE satisﬁes these requirements.

SPECTRE成就的关键在于它只需满足比传统共识所要求的更弱的属性。 
在传统模型中，任何两个交易之间的顺序必须由所有非损坏节点决定并同意。 
相反，SPECTRE只满足诚实用户进行的交易。 
我们观察到，在货币的情况下，同时发布的两个冲突的支付只能由不诚实的用户创建，因此我们可以承担延迟接受这种交易而不损害系统的可用性。 
我们的框架先形式化定义了加密货币的分布式账本所需的弱要求的集合。 
然后我们提供一个SPECTRE满足这些要求的形式化证明。
