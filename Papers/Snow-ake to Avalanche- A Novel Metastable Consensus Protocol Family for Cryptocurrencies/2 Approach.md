## 2 Approach

We start with a non-Byzantine protocol, Slush, and progressively build up Snowflake, Snowball and Avalanche, all based on the same common metastable mechanism.
Though we provide definitions for the protocols, we defer their formal analysis and proofs of their properties to the next section.

Overall, this protocol family achieves its properties by humbly cheating in three different ways. 
First, taking inspiration from Bitcoin, we adopt a safety guarantee that is probabilistic. 
This probabilistic guarantee is indistinguishable from traditional safety guarantees in practice, since appropriately small choices of $\epsilon $ can render consensus failure practically infeasible, less frequent than CPU miscomputations or hash collisions. 
Second, instead of a single replicated state machine (RSM) model, where the system determines a sequence of totally-ordered trans- actions $T_0$, $T_1$, $T_2$, ...issued by any client, 
we adopt a parallel consensus model with authenticated clients, where each client interacts independently with its own RSMs and optionally transfers ownership of its RSM to another client. 
The system establishes only a partial order between dependent transactions. 
Finally, we provide no liveness guarantee for misbehaving clients, but ensure that well-behaved clients will eventually be serviced. 
These techniques, in conjunction, enable the system to nevertheless implement a useful Bitcoin-like cryptocurrency, with drastically better performance and scalability.

2.1 Model, Goals, Threat Model

We assume a collection of nodes, $N$ , composed of correct nodes $C$ and Byzantine nodes $B$, where $n = |N|$. 
We adopt what is commonly known as Bitcoin’s unspent transaction output (UTXO) model. 
In this model, clients are authenticated and issue cryptographically signed transactions that fully consume an existing UTXO and issue new UTXOs. 
Unlike nodes, clients do not participate in the decision process, but only supply transactions to the nodes running the consensus protocol. 
Two transactions conflict if they consume the same UTXO and yield different outputs. 
Correct clients never issue conflicting transactions, nor is it possible for Byzantine clients to forge conflicts with transactions issued by correct clients.
However, Byzantine clients can issue multiple transactions that conflict with one another, and correct clients can only consume one of those transactions. 
The goal of our family of consensus protocols, then, is to accept a set of non-conflicting transactions in the presence of Byzantine behavior. 
Each client can be considered as a replicated state machine whose transitions are defined by a totally ordered list of accepted transactions.

Our family of protocols provide the following guaran- tees with high probability:

> P1. Safety.   
> No two correct nodes will accept conflicting transactions.
> 
> P2. Liveness.   
> Any transaction issued by a correct client (aka virtuous transaction) will eventually be accepted by every correct node.  

Instead of unconditional agreement, our approach guarantees that safety will be upheld with probability $1 − \epsilon $, 
where the choice of the security parameter $\epsilon $ is under the control of the system designer and applications.

We assume a powerful adaptive adversary capable of observing the internal state and communications of every node in the network,  
but not capable of interfering with communication between correct nodes.   
Our analysis assumes a synchronous network, while our deployment and evaluation is performed in a partially synchronous setting.   
We conjecture that our results hold in partially syn- chronous networks, but the proof is left to future work.   
We do not assume that all members of the network are known to all participants,   
but rather may temporarily have some discrepancies in network view.   
We assume a safe bootstrapping system, similar to that of Bitcoin,   
that enables a node to connect with sufficiently many correct nodes to acquire a statistically unbiased view of the network.   
We do not assume a PKI.   
We make standard cryptographic assumptions related to public key signatures and hash functions.

2.2 Slush: Introducing Metastability

The core of our approach is a single-decree consensus protocol, inspired by epidemic or gossip protocols.   
The simplest metastable protocol, Slush, is the foundation of this family, shown in Figure 1.   
Slush is not tolerant to Byzantine faults, but serves as an illustration for the BFT protocols that follow.  
For ease of exposition, we will describe the operation of Slush using a decision between two conflicting colors, red and blue.

<img width="401" alt="2018-09-27 9 30 01" src="https://user-images.githubusercontent.com/39436379/46149299-94af4000-c29c-11e8-96c8-11fe9367b718.png">
Figure 1: Slush protocol. Timeouts elided for readability.

In Slush, a node starts out initially in an uncolored state.   
Upon receiving a transaction from a client, an uncolored node updates its own color to the one carried in the transaction and initiates a query.   
To perform a query, a node picks a small, constant sized ($k$) sample of the network uniformly at random, and sends a query message.   
Upon receiving a query, an uncolored node adopts the color in the query, responds with that color,   
and initiates its own query, whereas a colored node simply responds with its current color.   
If $k$ responses are not received within a time bound, the node picks an additional sample from the remaining nodes uniformly at random and queries them until it collects all responses.   
Once the querying node collects $k$ responses, it checks if a fraction $\geq \alpha k$ are for the same color, where $\alpha > 0.5$ is a protocol parameter.   
If the $\alpha k$ threshold is met and the sampled color differs from the node’s own color, the node flips to that color.   
It then goes back to the query step, and initiates a subsequent round of query, for a total of $m$ rounds.   
Finally, the node decides the color it ended up with at time $m$.

This simple protocol has some curious properties.  
First, it is almost $memoryless$:   
a node retains no state between rounds other than its current color, and in particular maintains no history of interactions with other peers.   
Second, unlike traditional consensus protocols that query every participant,   
every round involves sampling just a small, constant-sized slice of the network at random.   
Second, even if the network starts in the metastable state of a 50/50 red-blue split,   
random perturbations in sampling will cause one color to gain a slight edge and repeated samplings afterwards will build upon and amplify that imbalance.   
Finally, if $m$ is chosen high enough, Slush ensures that all nodes will be colored identically whp.   
Each node has a constant, predictable communication overhead per round, and we will show that $m$ grows logarithmically with $n$.

<img width="401" alt="2018-09-27 9 30 01" src="https://user-images.githubusercontent.com/39436379/46150615-80b90d80-c29f-11e8-9beb-e438640ce049.png">

If Slush is deployed in a network with Byzantine nodes, the adversary can interfere with decisions.   
In particular, if the correct nodes develop a preference for one color,   
the adversary can attempt to flip nodes to the opposite so as to keep the network in balance.   
The Slush protocol lends itself to analysis but does not, by itself,   
provide a strong safety guarantee in the presence of Byzantine nodes,   
because the nodes lack state. We address this in our first BFT protocol.

2.3 Snowflake: BFT
Snowflake augments Slush with a single counter that captures the strength of a node’s conviction in its current color.   
This per-node counter stores how many consecutive samples of the network have all yielded the same color.   
A node accepts the current color when its counter exceeds $\beta $, another security parameter.   
Figure 2 shows the amended protocol, which includes the following modifications:
1. Each node maintains a counter $cnt$;
2. Upon every color change, the node resets $cnt$ to 0;
3. Upon every successful query that yields $\geq \alpha k$ responses for the same color as the node, the node increments $cnt$.
When the protocol is correctly parameterized for a given threshold of Byzantine nodes and a desired $\epsilon -$ guarantee, it can ensure both safety (P1) and liveness (P2).   
As we later show, there exists a phase-shift point after which correct nodes are more likely to tend towards a decision than a bivalent state.   
Further, there exists a point-of-no-return after which a decision is inevitable.   
The Byzantine nodes lose control past the phase shift, and the correct nodes begin to commit past the point-of-no-return, to adopt the same color, whp.

2.4 Snowball: Adding Confidence

Snowflake’s notion of state is ephemeral: 
the counter gets reset with every color flip. While, theoretically, the protocol is able to make strong guarantees with minimal state we
tack by incorporating a more permanent notion of belief. Snowball augments Snowflake with confidence counters that capture the number of queries that have yielded a threshold result for their corresponding color (Figure 3). A node decides a color if, during a certain number of consecutive queries, its confidence for that color exceeds that of other colors. The differences between Snowflake and Snowball are as follows:
1. Upon every successful query, the node increments its confidence counter for that color.  
2. A node switches colors when the confidence in its current color becomes lower than the confidence value of the new color.  

Snowball is not only harder to attack than Snowflake, but is more easily generalized to multi-decree protocols.

<img width="357" alt="2018-09-28 4 15 13" src="https://user-images.githubusercontent.com/39436379/46196516-eb1e8c00-c339-11e8-9b30-fe8a78117346.png">
  
Figure 4: Avalanche: the main loop.

2.5 Avalanche: Adding a DAG

Avalanche, our final protocol, generalizes Snowball and maintains a dynamic append-only Directed Acyclic Graph (DAG) of all known transactions.   
The DAG has a single sink that is the *genesis vertex*.   
Maintaining a DAG provides two significant benefits.   
First, it improves effi- ciency, because a single vote on a DAG vertex implicitly votes for all transactions on the path to the genesis vertex.   
Second, it also improves security, because the DAG intertwines the fate of transactions, similar to the Bitcoin blockchain.   
This renders past decisions difficult to undo without the approval of correct nodes.

When a client creates a transaction, it names one or more *parents*, which are included inseparably in the transaction and form the edges of the DAG.   
The parent-child relationships encoded in the DAG may, but do not need to, correspond to application-specific dependencies;   
for instance, a child transaction need not spend or have any relationship with the funds received in the parent transaction.   
We use the term *ancestor set* to refer to all transactions reachable via parent *edges* back in history, and *progeny* to refer to all children transactions and their off-spring.

The central challenge in the maintenance of the DAG is to choose among conflicting transactions.   
The notion of conflict is application-defined and transitive, forming an equivalence relation.   
In our cryptocurrency application, transactions that spend the same funds (*double-spends*) conflict, and form a conflict set, out of which only a single one can be accepted.   
Note that the conflict set of a virtuous transaction is always a singleton.   
Shaded regions in Figure 7 indicate conflict sets.  

Avalanche embodies a Snowball instance for each conflict set.   
Whereas Snowball uses repeated queries and multiple counters to capture the amount of confidence built in conflicting transactions (colors), Avalanche takes advantage of the DAG structure and uses a transaction’s progeny.   
Specifically, when a transaction *T* is queried, all transactions reachable from *T* by following the DAG edges are implicitly part of the query.   
A node will only respond positively to the query if T and its entire ancestry are currently the preferred option in their respective conflict sets.   
If more than a threshold of responders vote positively, the transaction is said to collect a chit, $C_{uT}= 1$, otherwise, $C_{uT}= 0$.   
Nodes then compute their *confidence* as the sum of chit values in the progeny of that transaction.   
Nodes query a transaction just once and rely on new vertices and chits, added to the progeny, to build up their confidence.   
Ties are broken by an initial preference for first-seen transactions.

2.6 Avalanche: Specification
Each correct node $u$ keeps track of all transactions it has learned about in set $\tau _u$, partitioned into mutually exclusive conflict sets $P_T$, $T /in T_u$. Since conflicts are transitive, if $T_i$ and $T_j$ are conflicting, then $P_{T_i}=P_{T_j}$.
We write ${T}'\leftarrow T$ if $T$ has a parent edge to transaction ${T}'$, The “$\overset{*}{\leftarrow}$”-relation is its reflexive transitive closure, indicating a path from $T$ to ${T}'$. Each node $u$ can compute a confidence value, $d_u(T)$, from the progeny as follows:
$d_u(T)=\sum_{{T}'\in \tau _u, T\overset{*}{\leftarrow}{T}'}{C_{u{T}'}}$

<img width="443" alt="2018-09-28 4 36 50" src="https://user-images.githubusercontent.com/39436379/46197555-c7107a00-c33c-11e8-8070-947af1f2a560.png">
Figure 6: Avalanche: voting and decision primitives.

In addition, it maintains its own local list of known nodes $N_u \subseteq N$ that comprise the system.   
For simplicity, we assume for now $N_u = N$ , and elide subscript $u$ in contexts without ambiguity.   
DAGs built by different nodes are guaranteed to be compatible.   
Specifically, if ${T}' \leftarrow T$, then every node in the system that has $T$ will also have ${T}'$ and the same relation ${T}' \leftarrow T$;   
and conversely, if ${T}' \not{{\leftarrow}} T$, then no nodes will end up with ${T}' \leftarrow T$.

Each node implements an event-driven state machine, centered around a query that serves both to solicit votes on each transaction and,   
simultaneously, to notify other nodes of the existence of newly discovered transactions.   
In particular, when node $u$ discovers a transaction $T$ through a query, it starts a one-time query process by sampling $k$ random peers.   
A query starts by adding $T$ to $\tau$,initializing $c_T$ to 0,and then sending a message to the selected peers.

Node u answers a query by checking whether each ${T}'$ such that $\overset{*}{\leftarrow}$ is currently preferred among competing transactions $\forall {T}'' /in P_{{T}'}$.   
If every single ancestor ${T}'$ fulfills this criterion, the transaction is said to be *strongly preferred*, and receives a yes-vote (1).   
A failure of this criterion at any ${T}'$ yields a no-vote (0).   
When u accumulates $k$ responses, it checks whether there are $\alpha k$ yes-votes for $T$, and if so grants the chit $c_T = 1$ for $T$.   
The above process will yield a labeling of the DAG with chit value $c_T$ and associated confidence for each transaction $T$.   
The chits are the result of one-time samples and are immutable, while $d(T)$ can increase as the DAG grows.   
Because $c_T$ values range from 0 to 1, confidence values are monotonic.

<img width="383" alt="2018-09-28 7 38 05" src="https://user-images.githubusercontent.com/39436379/46206299-1dd67d80-c356-11e8-8f26-5953603d9696.png">

Figure 7: Example of chit and confidence values, labeled as tuples in that order. Darker boxes indicate transactions with higher confidence values.

Figure 7 illustrates a sample DAG built by Avalanche. Similar to Snowball, sampling in Avalanche will create a positive feedback for the preference of a single transaction in its conflict set.   
For example, because $T_2$ has larger confidence than $T_3$, its descendants are more likely collect chits in the future compared to $T_3$.

Similar to Bitcoin, Avalanche leaves determining the acceptance point of a transaction to the application.   
An application supplies an **isAccepted** predicate that can take into account the value at risk in the transaction and the chances of a decision being reverted to determine when to decide.

Committing a transaction can be performed through a *safe early commitment*.   
For virtuous transactions, $T$ is accepted when it is the only transaction in its conflict set and has a confidence greater than threshold $\beta_1$.   
As in Snowball, T can also be accepted after a $\beta_2$ number of consecutive successful queries.   
If a virtuous transaction fails to get accepted due to a liveness problem with parents, it could be accepted if reissued with different parents.

Figure 5 shows how Avalanche performs parent selection and entangles transactions.   
Because transactions that consume and generate the same UTXO do not conflict with each other, any transaction can be reissued with different parents.

<img width="342" alt="2018-09-28 7 45 37" src="https://user-images.githubusercontent.com/39436379/46206615-1fed0c00-c357-11e8-9d92-d1b8f7483023.png">

Figure 4 illustrates the protocol main loop executed by each node.   
In each iteration, the node attempts to select a transaction $T$ that has not yet been queried.   
If no such transaction exists, the loop will stall until a new transaction is added to $\tau$.   
It then selects $k$ peers and queries those peers.   
If more than $\alpha k$ of those peers return a positive response, the chit value is set to 1.   
After that, it updates the preferred transaction of each conflict set of the transactions in its ancestry.   
Next, $T$ is added to the set $Q$ so it will never be queried again by the node.   
The code that selects additional peers if some of the $k$ peers are unresponsive is omitted for simplicity.  

Figure 6 shows what happens when a node receives a query for transaction $T$ from peer $j$.   
First it adds $T$ to $\tau$, unless it already has it.   
Then it determines if $T$ is currently strongly preferred.   
If so, the node returns a positive response to peer $j$.  
Otherwise, it returns a negative response.   
Notice that in the pseudocode, we assume when a node knows $T$, it also recursively knows the entire ancestry of $T$.   
This can be achieved by postponing the delivery of $T$ until its entire ancestry is recursively fetched.   
In practice, an additional gossip process that disseminates transactions is used in parallel, but is not shown in pseudocode for simplicity.
