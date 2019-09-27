# 2 From Trees to Directed Acyclic Graphs (DAGs)

We now begin to describe our proposed changes to the protocol. We start with a structural change to the blocks that will enable further modiﬁcations. In the current Bitcoin protocol, every block points at a single parent (via the parent’s hash), and due to natural (or malicious) forks in the network, the blocks form a tree.

We propose, instead, the node creating the block would list all childless blocks that it was aware of. Surely, this added information does not hurt; it is simple to trace each of the references and see which one leads, for example, to the longest chain. We thus obtain a directed acyclic graph (DAG) in which each block references a subset of previous blocks. We assume that when block C references B, C’s creator knows all of B’s predecessors (it can request them). The information that can be extracted from a block’s reference list is suﬃcient to simulate the underlying chain selection rule: we can simulate the longest-chain rule, for example, by recursively selecting in each block a single link—the one leading to the longest chain.

The provision of this additional information amounts to a “direct revelation mechanism”: Instead of instructing nodes to select the chain they extend, we simply ask them to report all possible choices, and other nodes can simulate their choice, just as they would have made it (the term direct revelation is borrowed from economics where it is widely used in mechanism design [10]).

In fact, any chain selection protocol can be simulated in this manner, as the references provide all information needed to determine the choice that the block creator would have made when extending the chain. The only issue that needs to be handled is tie breaking (as in the case of conﬂicting chains of equal length). To do so, we ask nodes to list references to other blocks in some order, which is then used to break ties. Note that nodes are only required to list the childless nodes in the DAG; there is no need to list other nodes, as they are already reachable by simply following the links. <sup>5</sup>

> <sup>5</sup> DAGs are already required by GHOST (although for diﬀerent reasons), and Ethereum’s blocks currently reference parent blocks as well as “uncles” (blocks that share the same parent as their parent). Thus, this modiﬁcation is quite natural.

Formally, we denote by BDAG the set of all directed acyclic block graphs G = (V, E) with vertices V (blocks) and directed edges E, where each B ∈ V has in addition an order ≺ B over all its outgoing edges. In our setup, an edge goes from a block to its parent, thus childless vertices (“leaves”) are those with no incoming edges. Graphs in BDAG are required to have a unique maximal vertex, “the genesis block”. We further denote by sub(B, G) the subgraph that includes all blocks in G reachable from B.

An underlying chain selection rule F is used to decide on the main chain in the DAG (e.g., longest-chain or GHOST). The rule F is a mapping from block DAGs to block chains such that for any G ∈ BDAG, F(G) is a maximal (i.e., non-extendable) chain in G. The order ≺ B is assumed to agree with F, in the sense that if A is one of B’s parents and A ∈ F(sub(B, G)), then A is ﬁrst in the order ≺ B .
2.1 Exploiting the DAG Structure—The Inclusive Protocol

We deﬁne Inclusive-F, the “Inclusive” version of the chain selection rule F, which incorporates non-conﬂicting oﬀ-chain transactions into a given blocks accepted transaction set. Intuitively, a block B uses a postorder traversal on the block DAG to form a linear order on all blocks. If two conﬂicting transactions appear, the one that appeared earlier according to this order is considered to be the one that has taken place (given that all previous transactions it depends on have also occurred). Thus, we use the order on links that blocks provide to deﬁne an order on blocks, which we then use to order transactions that appear in those blocks, and ﬁnally, we conﬁrm transactions according to this order.

To make the Inclusive algorithm formal, we need to provide a method to decide precisely the set of accepted transactions. Bitcoin transactions are composed of inputs (sources of funds) and outputs (the targets of funds). Outputs are, in turn, spent by inputs that redirect the funds further. We deﬁne the consistency of a transaction set, and its maximality as follows:

Deﬁnition 1. Given a set of transactions T, a transaction tx is consistent with T if all its inputs are outputs of transactions in T, and no other transaction in T uses them as inputs. We say that T is consistent, if every transaction tx ∈ T is consistent with T \ {tx}.

Deﬁnition 2. We say that a consistent set of transactions T from a block DAG G is maximal, if no other consistent set T ′ of transactions from G contains T.
The algorithm below performs a postorder traversal of the DAG sub(B, G). Along its run it conﬁrms any transaction that is consistent with those accepted thus far. The traversal backtracks if it visits the same block twice. <sup>6</sup>

<sup>6</sup> It is important to note that the algorithm below describes a full traversal. More eﬃcient implementations are possible if a previously traversed DAG is merely being updated (with methods similar to the unspent transaction set used in Bitcoin).

If B is the genesis block, which has no predecessors, m = 0.

The algorithm is to be called with arguments Inclusive-F(G, B, ∅), initially setting visited(·) as False for all blocks. Its output is the set of transactions it approves.

Algorithm 1. Inclusive-F(G, B, T) Input: a DAG G, a block B with pointers to predecessors (B 1 , ..., B m ) (ordered according to ≺ B ), <sup>7</sup> and a set of previously conﬁrmed transactions T.

> <sup>7</sup> If B is the genesis block, which has no predecessors, m = 0.

1. IF visited(B) RETURN T

2. SET visited(B):=True

3. FOR i = 1 TO m:

4. T = Inclusive-F(G, B i , T)

5. FOR EACH tx ∈ B

6. IF (tx is consistent with T) THEN T = T ∪ {tx}

7. RETURN T

We say that B is a valid block if at the end of the run on sub(B, G) we have B ⊆ T.  <sup>8</sup>  The algorithm’s run extends ≺ B to a linear order on sub(B, G), deﬁned by: A ≺ B A ′ if Inclusive-F(G, B, ∅) visited A before it visited A ′ . The following proposition states that the algorithm provides consistent and maximal transaction sets:

> <sup>8</sup> The Inclusive algorithm can also handle blocks that have some of their transactions rejected.

Proposition 1. Let T be the set returned by Inclusive-F(G, B, ∅). Then T is both consistent and maximal in sub(B, G).

The proof is immediate from the algorithm.

An important property of this protocol is that once a transaction has been approved by some main chain block B of G, it will remain in the approved set of any extending block as long as B remains in G’s main chain. This is because transactions conﬁrmed by main chain blocks are ﬁrst to be included in the accepted transaction sets of future main chain blocks. Since both in longest-chain and GHOST blocks that are buried deep in the main chain become increasingly less likely to be replaced, the same security guarantees hold for transactions included in their Inclusive versions.

Fees and Rewards Each transaction awards a fee to the creator of the ﬁrst block that included it in the set T. Formally, let A be some block in sub(B, G). Denote by T(A) the set of transactions which block A was the ﬁrst to contain, according to the order ≺ B . Then (according to B’s world view) A’s creator is awarded a fraction of the fee from every tx ∈ T(A). Although na¨ıvely we would want to grant A all of T(A)’s fees, security objectives cannot always permit it. This is one of the main tradeoﬀs in the protocol: On the one hand, we wish to award fees to anyone that included a new transaction. This implies that poorly connected miners that were slow to publish their block will still receive rewards. On the other hand, oﬀ-chain blocks may also be the result of malicious action, including published blocks from a failed double-spend attack. In this case we would prefer no payoﬀ would be received. We therefore allow for a somewhat tolerant payment mechanism that grants a block A a fraction of the reward which depends on how quickly the block was referenced by the main chain. The analysis that will follow (in Sect. 3) will justify the need for lower payments.

Formally, for any block A ∈ G deﬁne by pre(A) the latest main chain block which is reachable from A, and by post(A) the earliest main chain block from which A is reachable; if no such block exists, regard post(A) as a “virtual block” with height inﬁnity; if A is in the main chain then pre(A) = post(A) = A. Denote c(A) := post(A).height − pre(A).height; c(·) is a measure of the delay in a block’s publication (with respect to the main chain).

In order to penalize a block according to its gap parameter c(·) we make use of a generic discount function, denoted γ, which satisﬁes: γ : N ∪ {0} → [0, 1], it is weakly decreasing, and γ(0) = 1. The payment for (the creator of) block A is deﬁned by:

γ (c(A)) · v(w),

w∈T (A)

where v(w) is the fee of transaction w. In other words, A gains only a fraction γ(c(A)) of its original rewards. By way of illustration, consider the following discount function:

Example 1.

γ 0 (c) =

1

10−c

7

0

0≤c≤3 3 < c < 10 c ≥ 10

(1)

γ 0 grants a full reward to blocks which are adequately synchronized with the main chain (γ 0 (c) = 1 for c ≤ 3), on the one hand, and pays no reward at all to blocks that were left “unseen” by the main chain for too long, on the other hand (γ 0 (c) = 0 for c ≥ 10); in the mid-range, a block is given some fraction of the transaction rewards (γ 0 (c) = 10−c 7 for 3 < c < 10).

Money Creation In addition to fees, Bitcoin and other cryptocurrencies use the block creation process to create and distribute new coins. Newly minted coins can also be awarded to oﬀ-chain blocks in a similar fashion to transaction fees, i.e., in amounts that decrease for blocks that were not quickly included in the main chain. A block’s reward can therefore be set as a fraction γ(c(A)) of the full reward on the chain.  <sup>9</sup>  As our primary focus is on the choice of transactions

> <sup>9</sup> The total reward can be automatically adjusted to maintain a desired rate of money creation by a process similar to the re-targeting done for diﬃculty adjustments.

to include in the block, we assume for simplicity from this point on, that no money creation takes place (i.e., that money creation has decayed to negligible amounts—as will eventually occur for Bitcoin).

Now that we have deﬁned the Inclusive protocol, we begin to analyze its implications.

# References
[10]. Nisan, N., Roughgarden, T., Tardos, E., Vazirani, V.V.: Algorithmic game theory, chap. 9. Cambridge University Press (2007)