> Source: https://eprint.iacr.org/2019/611.pdf
# 4 Dynamic Accumulator
# 4 动态累加器

We introduce a hash-based dynamic accumulator with no trusted setup or manager requirements.
Introduced in [6], accumulators are compact representations of a set, to which elements can be added and proven.
Our accumulator uses a forest of perfect Merkle trees [7] and extends the work of [8] to allow eﬃcient removals of elements from the accumulator, reducing the total number of leaves in the forest when deletions occur.

Additions are computable without any data beyond the accumulator and the element to be added, and deletions are computable with an inclusion proof of the data to be deleted.

The design of the accumulator is a forest of perfect binary hash trees.
The representation of the accumulator which must be stored includes: the number of elements stored, and the root of every tree in the forest.

To update the accumulator, Add(), Delete() and Verify() functions are deﬁned, which each operate on single elements.
Batched operations, where multiple elements are added or removed at the same time can speed up operations and reduce the sizes of proofs.

## 4.1 Logical structure of the binary forest

We arrange the elements of the accumulator into a forest of perfect binary trees with the largest tree on the left and smallest on the right.
This arrangement allows for a more intuitive visualization of trees merging and splitting when needed.
Row operations are also possible where elements can move between sub-trees.

 As the trees in the forest are always perfect, they hold 2 h leaves, where h is the height of the tree.
Any natural number of leaves can be organized into a forest of perfect binary trees, just as any natural number can be represented by a sum of powers of two.
This relation provides a convenient shortcut: the number of trees in the accumulator is the number of 1-bits in the binary representation of the number of leaves.
The heights of those trees is the bit-position of the 1-bits in that representation.
For example, a forest with 133 leaves would have 3 trees: a height 7 tree, a height 2 tree, and a height 0 tree.
This is quickly visible by looking at the binary representation of the number 133: 10000101.

Any set of leaves can be grouped into binary trees using this method.
In all cases, it is possible to add one more leaf to the forest knowing only the roots of each tree.
In the 133 leaf example, adding an extra leaf would result in 134 leaves, with a binary representation of 10000110.
The 0-height tree (which itself is a leaf) would combine with the newly added leaf to create a 1-height tree with 2 leaves.
A further addition to 135 would then create a 0-height tree with the additional leaf.

## 4.2 Adding and removing elements

We describe here how to add or remove a single element, which suﬃces for all operations as these algorithms can be invoked many times to add or remove many elements.
In the case of removing elements, batching many removals into a single operation can signiﬁcantly reduce CPU usage.
The batch operation is described with examples in the appendix.

The Parent() function, used below, is the typical concatenate and hash function from Merkle trees; we append the height within the tree to prevent attacks such as in [9].
To simplify the pseudocode, this height argument is left out of the Parent() function call, as well as the left / right information in DeleteOne(), which can be obtained from the proof argument.

We represent the accumulator’s Merkle forest roots as an array of hashes, which can include empty hashes.
acc[n] is the root at index n, or ∅ if that index is unpopulated (if there is no tree of that height).

### **Algorithm 1** AddOne

```vb
# add a leaf to the accumulator .
1: function AddOne(acc, leaf) 
# n is initially the leaf to add .
2:  n ← leaf 
# height is initially 0 .
3:  h ← 0 
# r is the lowest root .
4:  r ← acc[h] 
# loop until an empty root
5:  while r =6 ∅ do 
6:      n ← Parent(r, n) 
7:      acc[h] ← ∅ 
8:      h ← h + 1 
9:  r ← acc[h] 
10: acc[h] ← n 
11: return acc
```

.

AddOne() takes in the accumulator roots and an element to add.
It continues to compute parents until it encounters the ﬁrst unpopulated space in the accumulator array, at which point it stores the output of the ﬁnal parent function and returns a new list of hashes.
This new list can have one more populated hash (in the case where index 0 was empty), the same number, or fewer, down to a single element.

### **Algorithm 2** DeleteOne

```vb
# Delete leaf from the accumulator 
1: function DeleteOne(acc, proof) .
2:  n ← ∅ 
3:  h ← 0 
# Iterate over each proof element 
4:  while h < len(proof) do 
5:      p ← proof[h] .
6:       if n =6 ∅ then 
7:          n ← Parent(p, n) 
8:      else if acc[h] = ∅ then 
9:          acc[h] ← p 
10:     else 
11:         n ← Parent(p, acc[h]) 
12:         acc[h] ← ∅ 
13:     h ← h + 1 
14:     acc[h] ← n 
15:     return acc
```

DeleteOne() takes in the accumulator roots and an inclusion proof of the element to be deleted.
The inclusion proof is also a list of hashes, along with a position index indicating which proof elements are right and which are left Parent() arguments.
As in AddOne(), it begins with the smallest trees in the array, moving to larger trees, increasing height at each step.
The loop runs through every element of the proof, consuming a proof element at every step, and returning a modiﬁed array of roots when done.

There are two distinct phases of the ascending loop: breaking and hashing.
The deleted node can either be replaced with a tree root if one exists, or if one doesn’t the sibling of the deleted node (the proof node) is promoted into a tree itself.
Once a tree does exist and is swapped in to an empty spot, the algorithm latches into the hashing phase, where the remainder of the proofs are used to compute the root of the modiﬁed tree.

Thus the addition operation for the accumulator is to add elements, in whatever insertion order desired, to the bottom right of the forest.
Even without knowing the entire bottom row, the new roots can be calculated, and the newly added leaves can be forgotten after the addition has completed.

### 4.3 Combining veriﬁcation and deletion

Veriﬁcation and deletion require the same proof data.
This is convenient for our use case where elements are proven and then immediately deleted from the UTXO set.
An inclusion proof consists of a integer position of the element to be proven, and a sequence of siblings to insert into the parent function.
(Alternatively, a sequence of left / right ﬂags can be provided, but this is equivalent to indicating the position of the leaf to be proven.) An inclusion proof is considered valid if the ﬁnal hash computed is equal to the root already stored.
The proof is considered invalid otherwise.

Once an inclusion proof is deemed valid, that same proof can be used to delete the element from the accumulator.
This allows for an eﬃcient Verify / Delete combined function call which returns an error if the inclusion proof is invalid, and otherwise returns the modiﬁed accumulator with the element removed.

### 4.4 Bridge nodes

Using the above described accumulator, we can replace the on-disk database and store only the Merkle forest roots, while still adding and removing every TXO from the UTXO set.
The issue then becomes where these proofs come from.

Introducing the Utreexo accumulator design to an already running system presents challenges.
If we design a system with the accumulator in mind from the beginning, all wallet software which manages private keys would also manage and update inclusion proofs for UTXOs owned by that wallet.
However, the Bitcoin network has been in operation for over a decade, and a wide variety of software and hardware manage UTXOs, none of which has yet implemented the ideas detailed in this paper.
This poses a problem for the ﬁrst client, or compact state node, which implements this accumulator.
While a compact state node will be able to provide inclusion proofs for its own UTXOs, no other node will want them.
More critically, the compact state node will require inclusion proofs in order to verify every transaction it sees, but no other node will provide any proofs! Unless everyone coordinates and switches simultaneously, the compact state nodes will be left behind as soon as they start.

In order to simultaneously support old nodes which store the full UTXO set as well new nodes which use the accumulator, the network requires a “bridge node”.
A bridge node is a node which stores proofs for every UTXO in the accumulator.
In the case of Utreexo, a bridge node is simply a node which stores the entire Merkle forest at all times.

All nodes compute new hashes and update tree roots when additions and deletions to the set occur due to new blocks arriving.
As inclusion proofs consist of branches up to the roots, maintaining and updating proofs of every element in the set incurs no additional computational cost above computing the roots.
The only additional cost is space, as bridge nodes store approximately 2n hashes, in contrast to the log(n) hashes stored by compact state nodes.

In order for the bridge node to produce proofs with minimal latency, in addition to the full forest, a mapping of TXO identiﬁer to leaf position must also be maintained.
As the leaves in the forest are unsorted and in fact shuﬄe positions during accumulator updates, a bridge node would need to search linearly through the set of leaves to ﬁnd a proof without such an index.
A lookup table mapping outpoints to leaf positions presents an additional space requirement but improves speed.
Ideally, a single bridge node should be able to attach proofs and relay all transactions to the network of compact state nodes with minimal latency.

### 4.5 Network Design

Compact state nodes can be incrementally added to the current network with no chaggnges to the existing node software.
Full nodes operate as before, propagating transactions and blocks to each other.
The bridge node is a full node which also stores the entire Merkle forest and can provide proofs for compact state nodes.
The bridge node does not announce itself as such; to full nodes, it appears to be a standard full node, and to compact state nodes it appears to be a compact state node.
While the bridge node links the two networks together, it is only needed to bridge in one direction: from the existing full node network to the Utreexo network.
While the bridge node can send transactions from the Utreexo network to the full node network, so can any compact state node, as the data sent in the Utreexo network is a super-set of the data needed by full nodes.


![Figure 1](./Figures/1.jpg)

>Figure 1: A bridge node connects the network of already existing full nodes to the network of compact state nodes.
Full nodes propagate transactions (and blocks of transactions) to each other.
Compact state nodes similarly propagate transactions, and also send inclusion proofs along with every transaction.
Compact state nodes can send transaction messages to full nodes by omitting the inclusion proofs, but cannot receive transactions directly from full nodes, which are unable to provide proofs.

### 4.6 Full and partial forest storage

Bridge nodes need to store the entirety of the hash forest data.
An eﬃcient storage mechanism for this is to store every hash sequentially in memory, and compute byte oﬀsets to seek to a speciﬁc hash.
This has no space overhead, but some I/O overhead when swapping elements.
This can be implemented with a linear array of hashes, some of which remain unpopulated.
We have also implemented the mapping of UTXO identiﬁers to leaf position in levelDB, so that bridge nodes can quickly provide an inclusion proof when they receive a transaction.

Compact state nodes need only store the tree roots, but as detailed in the next section, are able to trade some additional storage for reduced download size.
To store partial forests, we use a variant of pointer-based binary trees where every node has two pointers to cousin nodes (the children of a node’s sibling) rather than directly pointing to children.
This allows eﬃcient storage and processing of subsets of the forest, from the roots only to the entire forest (though if storing the entire forest, using a hash array is more eﬃcient as no pointers are needed.)

# Reference
[6] Josh Benaloh and Michael de Mare. One-way accumulators: A decentralized alternative to digital sinatures (extended abstract). In EUROCRYPT, 1993.

[7] Ralph C. Merkle. Protocols for public key cryptosystems. 1980 IEEE Symposium on Security and Privacy, pages 122–122, 1980.

[8] Leonid Reyzin and Sophia Yakoubov. Eﬃcient asynchronous accumulators for distributed pki. Cryptology ePrint Archive, Report 2015/718, 2015. https://eprint.iacr.org/2015/718.

[9] Charles Bouillaguet, Pierre-Alain Fouque, Adi Shamir, and Sebastien Zimmer. Second preimage attacks on dithered hash functions. Cryptology ePrint Archive, Report 2007/395, 2007. https://eprint.iacr. org/2007/395.