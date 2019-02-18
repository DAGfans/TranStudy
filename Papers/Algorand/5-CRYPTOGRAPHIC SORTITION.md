## 5 CRYPTOGRAPHIC SORTITION

## 5 密码学抽签

Cryptographic sortition is an algorithm for choosing a random subset of users according to per-user weights; that is, given a set of weights wi and the weight of all users W = Σi wi , the probability that user i is selected is proportional to w i /W . The randomness in the sortition algorithm comes from a publicly known random seed; we describe later how this seed is chosen. To allow a user to prove that they were chosen, sortition requires each user i to have a public/private key pair, (pk i ,sk i ). 

密码学抽签是一个根据每个用户的权重选择出一个随机子集用户的算法。详细说就是给出一个权重wi的集合，且所有用户的权重是W = Σi wi，那么用户i被选中的概率应该与wi/W成正比。在抽签算法中的随机性来源一个公众可知的随机数种子；我们后续会描述如何选择这个种子。为了让一个用户证明他被选中了，抽签要求每个用户i要有一个公/私钥对，记作(pki, ski)。

Sortition is implemented using verifiable random functions (VRFs) [39]. Informally, on any input string x, VRF sk (x) returns two values: a hash and a proof. The hash is a hashlenbit-long value that is uniquely determined by sk and x, but is indistinguishable from random to anyone that does not know sk. The proof π enables anyone that knows pk to check that the hash indeed corresponds to x, without having to know sk. For security, we require that the VRF provides these properties even if pk and sk are chosen by an attacker.

抽签是使用可验证随机函数(VRFs)[39]来实现。通俗的说，对于任何一个输入字符串x，VRF_sk(x)返回两个值：一个哈希(hash)和一个证明(proof)。hash是一个hash位长hashlenbit-lon的值，由sk和x唯一指定，(译注：“hash位长”指的是这个hash值作为bit的长度，例如一个hash值是4字节，那么总共就是4*8=64bit长)，但这对于任何不知道sk的人来说这个hash在随机性上是不可区分的。证明proof π 能让任何知道pk的人来检查这个hash是否真的和x对应，在不需要知道sk的情况下。为了安全性上讨论，我们要求VRF这些属性即使假设pk和sk被攻击选出。


### 5.1 Selection procedure

### 5.1 抽签过程

Using VRFs, Algorand implements cryptographic sortition as shown in Algorithm 1. Sortition requires a role parameter that distinguishes the different roles that a user may be selected for; for example, the user may be selected to propose a block in some round, or they may be selected to be the member of the committee at a certain step of BA⋆. Algorand specifies a threshold τ that determines the expected number of users selected for that role.

在使用VRFs下，Algorand 实现了加密抽签就像 Algorithm 1 中所展示的。抽签要求一个角色(role)参数来区分用户可能被选择的不同角色，例如用户可能在一些轮次(round)被选为出块者，或者其他的被选择为执行BA⋆的委员会成员，Algorand 指定一个阈值 τ 来决定这些角色成员预期的数量。

![image](https://user-images.githubusercontent.com/5023721/43693715-0f365c00-9962-11e8-8fb7-f6c06b3c5464.png)

Algorithm 1: The cryptographic sortition algorithm.

It is important that sortition selects users in proportion to their weight; otherwise, sortition would not defend against Sybil attacks. One subtle implication is that users may be chosen more than once by sortition (e.g., because they have a high weight). Sortition addresses this by returning the j parameter, which indicates how many times the user was chosen. Being chosen j times means that the user gets to participate as j different “sub-users.”

抽签根据用户的权益来抽取是重要的，否则抽签不能抵御女巫攻击(sybil attack)。一个微妙的需要注意的点是用户可能会被抽签多次抽中（例如，因为他们有高权重），解决方法是返回一个参数j，表面这个用户被选中的次数，被选择j次意味着用户可以作为j个不同的“子用户”参与。

To select users in proportion to their money, we consider each unit of Algorand as a different “sub-user.
” If user i owns w i (integral) units of Algorand, then simulated user (i,j) with j ∈ {1,...,w i } represents the j-th unit of currency  i owns, and is selected with probability p = τ/W , where W is the total amount of currency units in Algorand.

为了根据他们的钱来选择用户，我们认为 Algorand 的每一个单元都是一个不同的“子用户”。如果用户i拥有wi个单元的Algorand，那么被模拟的用户（i，j）j∈{1，... ，wi}代表着u用户i拥有的第j个货币单位，由概率p=τ/W 选择，W是总数。

As shown in Algorithm 1, a user performs sortition by computing ⟨hash,π⟩ ← VRF sk (seed||role), where sk is the user’s secret key. The pseudo-random hash determines how many sub-users are selected, as follows.
 The probability that exactly k out of the w (the user’s weight) sub-users are selected follows the binomial distribution, B(k;w,p) =(w k) p^k (1−p)^w−k , where Σ(w k=0) B(k;w,p) = 1. Since B(k1 ;n1 ,p) + B(k2 ;n2 ,p) = B(k1 + k2 ;n1 + n2 ,p), splitting a user’s weight (currency) among Sybils does not affect the number of selected sub-users under his/her control.

在算法1中展示的：一个用户展示抽签通过计算 ⟨hash,π⟩ ←VRFsk(seed||role)，其中sk是这个用户的私钥。 伪随机的hash决定了多少子用户被选中。确切的w分之k(w是用户的权益)个被选择的子用户的概率遵循二项分布，B(k;w,p) =(w k) p^k (1−p)^w−k，其中Σ(w, k=0) B(k;w,p) = 1。因为 B(k1 ;n1 ,p) + B(k2 ;n2 ,p) = B(k1 + k2 ;n1 + n2 ,p)，所以拆分用户的权益(钱)用户女巫攻击并不会影响由这个人控制的被选择的子用户数量(译注：这里的意思是说，使用二项分布的概率来解决子用户的选择，并不会随着实际控制人拆分自己的权益而对最终概率结果产生变化，因为B(k1 ;n1 ,p) + B(k2 ;n2 ,p) = B(k1 + k2 ;n1 + n2 ,p)，也就是说二项分布的和等于它们和的二项分布)

To determine how many of a user’s w sub-users are selected, the sortition algorithm divides the interval [0,1) into consecutive intervals of the form Ij = [Σ(j,k=0)B(k;w,p), Σ(j+1,k=0) B(k;w,p))  for j ∈ {0,1,...,w}. if hash/2_hashlen (where hashlen is the bit-length of hash) falls in the interval Ij , then the user has exactly j selected sub-users. The number of selected sub-users is publicly verifiable using the proof π (from the VRF output).

为了决定用户w有多少的子用户被选中，抽签算法把一个区间[0,1)分割成连续的区间，以Ij = [Σ(j,k=0)B(k;w,p), Σ(j+1,k=0) B(k;w,p))  for j ∈ {0,1,...,w}的形式组成，如果hash/2^hashlen(这里的hashlen就是hash的位bit长度)落在Ij的区间里，那么这个用户就是有j个子用户被选择出来，而被选出子用户数量能够使用 proof π 来被公开的验证。(π从VRF的输出中得到)

(译注：首先是`hash/2^hashlen`的解释。他表达的应该是这个意思：假设 hash 只有4位长，那么经过 VRF_sk 的hash应该是 0000-1111 其中的等概率的任何一个数字，然后`2^hashlen` 就是  `2^4=16 ==> 10000` 。所以这里的 `hash/2^hashlen` 的取值实际上就是0-1之间的任何一个概率值，取值在[0,1) 当中。对于 `[Σ(j,k=0) B(k; w,p), Σ(j+1,k=0) B(k; w,p))`  的解释：其中的`B(k; w, p)` 是二项分布的公式，注意共识里的求和是从k=0一直到k=j，而 j 是由每个while循环指定的。回顾一下二项分布的定义，而 `Σ(j,k=0) B(k; w,p)` 就是求 对于二项分布当概率为p的时候w的期望值。那么 `[Σ(j,k=0) B(k; w,p), Σ(j+1,k=0) B(k; w,p))`  就是这个概率落入期望的区间。这里我们注意：`p = τ/W`表面参与计算的概率p**对于每个人都是固定**的，由总体的权益大W和阈值τ决定，而影响每个人的j值的大小的决定性因素是本轮的随机到的抽签hash及每个人的权益小w。)

Sortition provides two important properties. First, given a random seed, the VRF outputs a pseudo-random hash value, which is essentially uniformly distributed between 0 and 2_hashlen −1. As a result, users are selected at random based on their weights. Second, an adversary that does not know ski cannot guess how many times user i is chosen, or if i was chosen at all (more precisely, the adversary cannot guess any better than just by randomly guessing based on the weights).

抽签提供了2个重要的属性，第一，给一个随机种子，VRF输出伪随机hash值，分布在0到2^hashlen-1的区间内，作为一个结果，用户依据他们权重被随机的选出，第2，一个攻击者不知道ski所以不能猜出用户i会被选中几次，或被选中(就是说攻击者只知道权重是不能猜出用户相关信息的)

The pseudocode for verifying a sortition proof, shown in Algorithm 2, follows the same structure to check if that user was selected (the weight of the user’s public key is obtained from the ledger). The function returns the number of selected sub-users (or zero if the user was not selected at all).

验证一个抽签证明使用算法2，使用同样的步骤来检查一个用户是否被选择出(用户公钥的权益可以从账本中找出)，这个函数返回用户被选中的子用户的数目(或者是0如果用户没有没选中)

![image](https://user-images.githubusercontent.com/5023721/43695062-f583ba6c-9968-11e8-81bf-efa09f681c22.png)


Algorithm 2: Pseudocode for verifying sortition of a user with public key pk.

### 5.2 Choosing the seed

### 5.2 选择一个种子

Sortition requires a seed that is chosen at random and publicly known. For Algorand, each round requires a seed that is publicly known by everyone for that round, but cannot be controlled by the adversary; otherwise, an adversary may be able to choose a seed that favors selection of corrupted users.

抽签要求一个公开的随机选取的种子，对于Algorand来说，每一轮都需要一个不被攻击者控制的且被所有人知道的种子，否则，攻击者可以选择一个有利于腐坏用户的种子。

In each round of Algorand a new seed is published. The seed published at Algorand’s round r is determined using VRFs with the seed of the previous round r −1. More specifically, during the block proposal stage of round r −1, every user u selected for block proposal also computes a proposed seed for round r as ⟨seed r ,π⟩ ← VRF sk u (seed r−1 ||r). Algorand requires that sku be chosen by u well in advance of the seed for that round being determined (§5.3). This ensures that even if u is malicious, the resulting seed r is pseudo-random.

在Algorand的每一轮中，都会发布一个新种子。 在Algorand的第r轮发表的种子是使用前一轮种子的VRFs确定的。更具体地，在r-1的区块提议阶段，每一个被选为区块提议的用户u同时也要为第r轮计算一个提议的种子作为 `<seed_r, π> <- VRF_sku(seed_r-1 || r)`。Algorand要求sku要在这一轮的种子选出前就得到 (§5.3)。这保证了即使u是攻击者，那么得到的seedr也是一个伪随机数。(译注：这里说的是sku必须在执行VRF之前就要存在，而不是针对这一轮的种子特点生成特定的私钥来产生对自己有利的种子。)

This seed (and the corresponding VRF proof π) is included in every proposed block, so that once Algorand reaches agreement on the block for round r −1, everyone knows seed r at the start of round r.
 If the block does not contain a valid seed (e.g., because the block was proposed by a malicious user and included invalid transactions), users treat the entire proposed block as if it were empty, and use a cryptographic hash function H (which we assume is a random oracle) to compute the associated seed for round r as seed r = H(seed r−1 ||r). The value of seed 0 , which bootstraps seed selection, can be chosen at random at the start of Algorand by the initial participants (after their public keys are declared) using distributed random number generation [14].

这个种子（伴随VRF的证明π）包含在每个提议的区块中，所以当Algorand达到第r-1轮次的区块时，所有人都会在r轮开始的时候就知道seed_r。如果这个区块没有一个包含一个有用的种子（比如这个区块由攻击者发出包含非法的交易），用户就把这个提议的区块看作空，然后用一个密码学的哈希函数H（H我们假定是一个随机数oracle）来计算出对于轮次r的相关种子 seed_r = H(seed_r-1||r)。seed_0是初始种子，在Algorand启动的时候选择出(在公钥都已经宣告后)，使用一个随机数生成器得到。

To limit the adversary’s ability to manipulate sortition, and thus manipulate the selection of users for different committees, the selection seed (passed to Algorithm 1 and Algorithm 2) is refreshed once every R rounds. Namely, at roundr Algorand calls the sortition functions with seed r−1−(r mod R) .

为了限制攻击者有意操作抽签，选举种子（传给算法1和算法2）每R轮刷新一次，具体的说，在第r轮Algorand调用抽签的种子是 seed_(r-1-(r mod R))

### 5.3 Choosing sk u well in advance of the seed

### 5.3 在种子之前就选好 sku

Computing seed r requires that every user’s secret key sk u is chosen well in advance of the selection seed used in that round, i.e., seed r−1−(r mod R) . When the network is not strongly synchronous, the attacker has complete control over the links, and can therefore drop block proposals and force users to agree on empty blocks, such that future selection seeds can be computed. To mitigate such attacks Algorand relies on the weak synchrony assumption (in every period of length b, there must be a strongly synchronous period of length s < b). Whenever Algorand performs cryptographic sortition for round r, it checks the timestamp included in the agreed-upon block for round r −1−(r mod R), and uses the keys (and associated weights) from the last block that was created b-time before block r −1−(r mod R). The lower bound s on the length of a strongly synchronous period should allow for sufficiently many blocks to be created in order to ensure with overwhelming probability that at least one block was honest. This ensures that, as long as s is suitably large, an adversary u choosing a key sk u cannot predict the seed for round r.

要计算种子seed_r 要求每个用户的sku在这轮选出种子之前都准备好，例如seed r−1−(r mod R)。当网络不是强同步时，攻击者可以完全控制链路，因此可以丢弃阻塞提议并强制用户同意空块，从而可以计算未来的选择种子。(译注：这里是说攻击者在gossip的时候丢弃区块，这样其他正常用户会认为当前这轮出了个空快，而空块的计算在5.2中描述过，是统一的。所以此时攻击者就可以针对这个空块生成对自己有利的私钥。) 为了减轻在弱网络假设下的这种情况（在每个长度b的周期里面，一定要有一个强网络同步周期的长度s<b）。无论何时Algorand展现加密抽签在r轮的时候，它检查包含在r-1-(r mod R)轮的商定块中的时间戳，并使用在块r-1-(r-mod R)之前b-time创建的最后一个块的密钥（和相关权重）。这个下界s能够让强同步的时间中保证许多块按序创建来保证最后一个块是诚实的。这确保了只要s适合的大，攻击者u选择一个密钥sku是不能预测r轮的种子。

This look-back period b has the following trade-off: a large b mitigates the risk that attackers are able break the weak synchronicity assumption, yet it increases the chance that users have transferred their currency to someone else and therefore have nothing to lose if the system’s security breaks. This is colloquially known as the “nothing at stake” problem; one possible way to avoid this trade-off, which we do not explore in Algorand, is to take the minimum of a user’s current balance and the user’s balance from the look-back block as the user’s weight.

这种回溯期b有以下折衷：大b减轻了攻击者能够打破弱同步假设的风险，但它增加了用户将货币转移给其他人的机会，因此如果系统的安全性中断他们就什么也不会丢失。 这通俗地称为“nothing at stake”的问题; 避免这种折衷的一种可能的方式，我们不再Algorand中解释，即将用户的当前余额和来自回溯块的用户余额中的最小值作为用户的权重。


Appendix A formally analyzes the number of blocks that Algorand needs to be created in the period s when the network is strongly connected. We show that to ensure a small probability of failure F, the number of blocks is logarithmic in 1/F , which allows us to obtain high security with a reasonably low number of required blocks.

附录A形式化分析了在强网络环境下时，Algorand在周期s中需要创建的块的数量。我们证明为了确保失败F的可能性很小，块的数量在1/F中是对数的，这使我们能够以相当低的数量的所需块获得高安全性。 