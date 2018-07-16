4

The Algorithm

Our algorithm is a form of state machine replication [17, 34]: the service is modeled as a state machine that is replicated across different nodes in a distributed system. Each state machine replica maintains the service state and implements the service operations. We denote the set of replicas by and identify each replica using an integer in 0 1 . For simplicity, we assume

3 1 where is the maximum number of replicas that may be faulty; although there could be more than 3 1 replicas, the additional replicas degrade performance (since more and bigger messages are being exchanged) without providing improved resiliency.

The replicas move through a succession of conﬁgura- tions called views. In a view one replica is the primary and the others are backups. Views are numbered con- secutively. The primary of a view is replica such that

mod , where is the view number. View changes are carried out when it appears that the primary has failed. Viewstamped Replication [26] and Paxos [18]

used a similar approach to tolerate benign faults (as dis- cussed in Section 8.)

The algorithm works roughly as follows:

1. A client sends a request to invoke a service operation to the primary

2. The primary multicasts the request to the backups

3. Replicas execute the request and send a reply to the client

4. The client waits for 1 replies from different replicas with the same result; this is the result of the operation.

Like all state machine replication techniques [34], we impose two requirements on replicas: they must be deterministic (i.e., the execution of an operation in a given state and with a given set of arguments must always produce the same result) and they must start in the same state. Given these two requirements, the algorithm ensures the safety property by guaranteeing that all non- faulty replicas agree on a total order for the execution of requests despite failures.

The remainder of this section describes a simpliﬁed version of the algorithm. We omit discussion of how nodes recover from faults due to lack of space. We also omit details related to message retransmissions. Furthermore, we assume that message authentication is achieved using digital signatures rather than the more efﬁcient scheme based on message authentication codes; Section 5 discusses this issue further. A detailed formalization of the algorithm using the I/O automaton model [21] is presented in [4].

4.1 The Client A client requests the execution of state machine operation by sending a REQUEST message to the primary. Timestamp is used to ensure exactly- once semantics for the execution of client requests. Timestamps for ’s requests are totally ordered such that later requests have higher timestamps than earlier ones; for example, the timestamp could be the value of the client’s local clock when the request is issued.

Each message sent by the replicas to the client includes the current view number, allowing the client to track the view and hence the current primary. A client sends a request to what it believes is the current primary using a point-to-point message. The primary atomically multicaststhe request to all the backups using the protocol described in the next section.

A replica sends the reply to the request directly to the client. The reply has the form REPLY where is the current view number, is the timestamp of the corresponding request, is the replica number, and is the result of executing the requested operation.

The client waits for 1 replies with valid signatures from different replicas, and with the same and , before

3 accepting the result . This ensures that the result is valid, since at most replicas can be faulty.

If the client does not receive replies soon enough, it broadcasts the request to all replicas. If the request has already been processed, the replicas simply re-send the reply; replicas remember the last reply message they sent to each client. Otherwise, if the replica is not the primary, it relays the request to the primary. If the primary does not multicast the request to the group, it will eventually be suspected to be faulty by enough replicas to cause a view change.

In this paper we assume that the client waits for one request to complete before sending the next one. But we can allow a client to make asynchronous requests, yet preserve ordering constraints on them.

4.2 Normal-Case Operation

The state of each replica includes the state of the service, a message log containing messages the replica has accepted, and an integer denoting the replica’s current view. We describe how to truncate the log in Section 4.3.

When the primary, , receives a client request, , it starts a three-phase protocol to atomically multicast the request to the replicas. The primary starts the protocol immediately unless the number of messages for which the protocol is in progress exceeds a given maximum. In this case, it buffers the request. Buffered requests are multicast later as a group to cut down on message trafﬁc and CPUoverheads under heavy load;this optimization is similar to a group commit in transactional systems [11]. For simplicity, we ignore this optimization in the description below.

The three phases are pre-prepare, prepare, and commit. The pre-prepare and prepare phases are used to totally order requests sent in the same view even when the primary, which proposes the ordering of requests, is faulty. The prepare and commit phases are used to ensure that requests that commitare totally ordered across views.

In the pre-prepare phase, the primary assigns a sequence number, , to the request, multicasts a pre- prepare message with piggybacked to all the backups, and appends the message to its log. The message has the form PRE-PREPARE , where indicates the view in which the message is being sent, is the client’s request message, and is ’s digest.

Requests are not included in pre-prepare messages to keep them small. This is important because pre- prepare messages are used as a proof that the request was assigned sequence number in view in view changes. Additionally, it decouples the protocol to totally order requests from the protocol to transmit the request to the replicas; allowing us to use a transport optimized for small messages for protocol messages and a transport optimized for large messages for large requests.

A backup accepts a pre-prepare message provided:

the signatures in the request and the pre-prepare message are correct and is the digest for ; it is in view ; it has not accepted a pre-prepare message for view and sequence number containing a different digest; the sequence number in the pre-prepare message is between a low water mark, , and a high water mark, .

The last condition prevents a faulty primary from exhausting the space of sequence numbers by selecting a very large one. We discuss how and advance in Section 4.3.

If backup accepts the PRE-PREPARE message, it enters the prepare phase by multicasting a PREPARE message to all other replicas and adds both messages to its log. Otherwise, it does nothing.

A replica (including the primary) accepts prepare messages and adds them to its log provided their signatures are correct, their view number equals the replica’s current view, and their sequence number is between and .

We deﬁne the predicate prepared to be true if and only if replica has inserted in its log: the request

, a pre-prepare for in view with sequence number , and 2 prepares from different backups that match the pre-prepare. The replicas verify whether the prepares match the pre-prepare by checking that they have the same view, sequence number, and digest.

The pre-prepare and prepare phases of the algorithm guarantee that non-faulty replicas agree on a total order for the requests within a view. More precisely, they ensure the following invariant: if prepared is true then prepared is false for any non-faulty replica (including ) and any such that

. This is true because prepared and

3 1 imply that at least 1 non-faulty replicas have sent a pre-prepare or prepare for in view with sequence number . Thus, for prepared to be true at least one of these replicas needs to have sent two conﬂicting prepares (or pre-prepares if it is the primary for ), i.e., two prepares with the same view and sequence number and a different digest. But this is not possible because the replica is not faulty. Finally, our assumption about the strength of message digests ensures that the probability that and is negligible.

Replica multicasts a COMMIT to the other replicas when prepared becomes true. This starts the commit phase. Replicas accept commit messages and insert them in their log provided they are properly signed, the view number in the message is equal to the replica’s current view, and the sequence number is between and

4 We deﬁne the committed and committed-local predi- cates as follows: committed is true if and only if prepared is true for all in some set of

1 non-faulty replicas;and committed-local is true if and only if prepared is true and has accepted 2 1 commits (possibly including its own) from different replicas that match the pre-prepare for ; a commit matches a pre-prepare if they have the same view, sequence number, and digest.

The commit phase ensures the following invariant: if committed-local is true for some non-faulty

then committed is true. This invariant and the view-change protocol described in Section 4.4 ensure that non-faulty replicas agree on the sequence numbers of requests that commit locally even if they commit in different views at each replica. Furthermore, it ensures that any request that commits locally at a non-faulty replica will commit at 1 or more non-faulty replicas eventually.

Each replica executes the operation requested by

after committed-local is true and ’s state reﬂects the sequential execution of all requests with lower sequence numbers. This ensures that all non- faulty replicas execute requests in the same order as required to provide the safety property. After executing the requested operation, replicas send a reply to the client. Replicas discard requests whose timestamp is lower than the timestamp in the last reply they sent to the client to guarantee exactly-once semantics.

We do not rely on ordered message delivery, and therefore it is possible for a replica to commit requests out of order. This does not matter since it keeps the pre- prepare, prepare, and commit messages logged until the corresponding request can be executed.

Figure 1 shows the operation of the algorithm in the normal case of no primary faults. Replica0 isthe primary, replica 3 is faulty, and is the client.

request

pre-prepare

prepare

commit

reply

C 0 1 2 3

X

Figure 1: Normal Case Operation

4.3 Garbage Collection This section discusses the mechanism used to discard messages from the log. For the safety condition to hold, messagesmust be kept in a replica’s log until it knowsthat

the requests they concern have been executed by at least 1 non-faulty replicas and it can prove this to others in view changes. In addition, if some replica misses messages that were discarded by all non-faulty replicas, it will need to be brought up to date by transferring all or a portion of the service state. Therefore, replicas also need some proof that the state is correct.

Generating these proofs after executing every opera- tion would be expensive. Instead, they are generated periodically, when a request with a sequence number di- visible by some constant (e.g., 100) is executed. We will refer to the states produced by the execution of these re- quests as checkpoints and we will say that a checkpoint with a proof is a stable checkpoint.

A replica maintains several logical copies of the service state: the last stable checkpoint, zero or more checkpoints that are not stable, and a current state. Copy-on-write techniques can be used to reduce the space overhead to store the extra copies of the state, as discussed in Section 6.3.

The proof of correctness for a checkpoint is generated as follows. When a replica produces a checkpoint, it multicasts a message CHECKPOINT to the other replicas, where is the sequence number of the last request whose execution is reﬂected in the state and is the digest of the state. Each replica collects checkpoint messages in its log until it has 2 1 of them for sequence number with the same digest signed by different replicas (including possibly its own such message). These 2 1 messages are the proof of correctness for the checkpoint.

A checkpoint with a proof becomes stable and the replica discards all pre-prepare, prepare, and commit messages with sequence number less than or equal to

from its log; it also discards all earlier checkpoints and checkpoint messages.

Computing the proofs is efﬁcient because the digest can be computed using incremental cryptography [1] as discussed in Section 6.3, and proofs are generated rarely.

The checkpoint protocol is used to advance the low and high water marks (which limit what messages will be accepted). The low-water mark is equal to the sequence number of the last stable checkpoint. The high water mark , where is big enough so that replicas do not stall waiting for a checkpoint to become stable. For example, if checkpoints are taken every 100 requests, might be 200.

4.4 View Changes The view-change protocol provides liveness by allowing the system to make progress when the primary fails. View changes are triggered by timeouts that prevent backups from waiting indeﬁnitely for requests to execute. A backup iswaitingfor a request if it received a validrequest

5 and has not executed it. A backup starts a timer when it receives a request and the timer is not already running. It stops the timer when it is no longer waiting to execute the request, but restarts it if at that point it is waiting to execute some other request.

If the timer of backup expires in view , the backup starts a view change to move the system to view 1. It stops accepting messages (other than checkpoint, view-change, and new-view messages) and multicasts a VIEW-CHANGE 1 message to all replicas. Here is the sequence number of the last stable checkpoint known to , is a set of 2 1 valid checkpoint messages proving the correctness of , and

is a set containing a set for each request that prepared at with a sequence number higher than . Each set contains a valid pre-prepare message (without the corresponding client message) and 2 matching, valid prepare messages signed by different backups with the same view, sequence number, and the digest of .

When the primary of view 1 receives 2 valid view-change messages for view 1 from other replicas, it multicasts a NEW-VIEW 1 message to all other replicas, where is a set containing the valid view- change messages received by the primary plus the view- change message for 1 the primary sent (or would have sent), and is a set of pre-prepare messages (without the piggybacked request). is computed as follows:

1. The primary determines the sequence number min-s of the latest stable checkpoint in and the highest sequence number max-s in a prepare message in .

2. The primary creates a new pre-prepare message for view 1 for each sequence number between min-s and max-s. There are two cases: (1) there is at least one set in the component of some view-change message in with sequence number , or (2) there is no such set. In the ﬁrst case, the primary creates a new message PRE-PREPARE 1 , where is the request digest in the pre-prepare message for sequence number with the highest view number in . In the second case, it creates a new pre- prepare message PRE-PREPARE 1 , where is the digest of a special null request; a null request goes through the protocol like other requests, but its execution is a no-op. (Paxos [18] used a similar technique to ﬁll in gaps.)

Next the primary appends the messages in to its log. If min-s is greater than the sequence number of its latest stable checkpoint, the primary also inserts the proof of stability for the checkpoint with sequence number min-s in its log, and discards information from the log as discussed in Section 4.3. Then it enters view 1: at this point it is able to accept messages for view 1.

A backup accepts a new-view message for view 1 if it is signed properly, if the view-change messages it

contains are valid for view 1, and if the set is correct; it veriﬁes the correctness of by performing a computation similar to the one used by the primary to create . Then it adds the new information to its log as described for the primary, multicasts a prepare for each message in to all the other replicas, adds these prepares to its log, and enters view 1.

Thereafter, the protocol proceeds as described in Section 4.2. Replicas redo the protocol for messages between min-s and max-s but they avoid re-executing client requests (by using their stored information about the last reply sent to each client).

A replica may be missing some request message or a stable checkpoint (since these are not sent in new- view messages.) It can obtain missing information from another replica. For example, replica can obtain a missing checkpoint state from one of the replicas whose checkpoint messages certiﬁed its correctness in . Since 1 of those replicas are correct, replica will always obtain or a later certiﬁed stable checkpoint. We can avoid sending the entire checkpoint by partitioning the state and stamping each partition with the sequence number of the last request that modiﬁed it. To bring a replica up to date, it is only necessary to send it the partitions where it is out of date, rather than the whole checkpoint.

4.5 Correctness This section sketches the proof that the algorithm provides safety and liveness; details can be found in [4].

4.5.1 Safety As discussed earlier, the algorithm provides safety if all non-faulty replicas agree on the sequence numbers of requests that commit locally.

In Section 4.2, we showed that if prepared is true, prepared is false for any non-faulty replica (including ) and any such that . This implies that two non-faulty replicas agree on the sequence number of requests that commit locally in the same view at the two replicas.

The view-change protocol ensures that non-faulty replicas also agree on the sequence number of requests that commit locally in different views at different replicas. A request commits locally at a non-faulty replica with sequence number in view only if committed is true. This means that there is a set 1 containing at least 1 non-faulty replicas such that prepared is true for every replica in the set.

Non-faulty replicas will not accept a pre-prepare for view withouthaving receiveda new-viewmessage for (since only at that point do they enter the view). But any correct new-view message for view contains correct view-change messages from every replica in a

6 set

of 2 1 replicas. Since there are 3 1 replicas, 1 and 2 must intersect in at least one replica that is not faulty. ’s view-change message will ensure that the fact that prepared in a previous view is propagated to subsequent views, unless the new-view message contains a view-change message with a stable checkpoint with a sequence number higher than . In the ﬁrst case, the algorithm redoes the three phases of the atomic multicast protocol for with the same sequence number and the new view number. This is important because it prevents any different request that was assigned the sequence number in a previous view from ever committing. In the second case no replica in the new view will accept any message with sequence number lower than . In either case, the replicas will agree on the request that commits locally with sequence number .

2

4.6 Non-Determinism

State machine replicas must be deterministic but many services involve some form of non-determinism. For example, the time-last-modiﬁed in NFS is set by reading the server’s local clock; if this were done independently at each replica, the states of non-faulty replicas would diverge. Therefore, some mechanism to ensure that all replicas select the same value is needed. In general, the client cannot select the value because it does not have enough information; for example, it does not know how its request will be ordered relative to concurrent requests by other clients. Instead, the primary needs to select the value either independently or based on values provided by the backups.

4.5.2 Liveness

To provide liveness, replicas must move to a new view if they are unable to execute a request. But it is important to maximize the period of time when at least 2 1 non-faulty replicas are in the same view, and to ensure that this period of time increases exponentially until some requested operation executes. We achieve these goals by three means.

First, to avoid starting a view change too soon, a replica that multicasts a view-change message for view 1 waits for 2 1 view-change messages for view 1 and then starts its timer to expire after some time . If the timer expires before it receives a valid new-view message for 1 or before it executes a request in the new view that it had not executed previously, it starts the view change for view 2 but this time it will wait 2 before starting a view change for view 3.

Second, if a replica receives a set of 1 valid view- change messages from other replicas for views greater than its current view, it sends a view-change message for the smallest view in the set, even if its timer has not expired; this prevents it from starting the next view change too late.

Third, faulty replicas are unable to impede progress by forcing frequent view changes. A faulty replica cannot cause a view change by sending a view-change message, because a view change will happen only if at least 1 replicas send view-change messages, but it can cause a view change when it is the primary (by not sending messages or sending bad messages). However, because the primary of view is the replica such that

mod , the primary cannot be faulty for more than consecutive views.

These three techniques guarantee liveness unless message delays grow faster than the timeout period indeﬁnitely, which is unlikely in a real system.

If the primary selects the non-deterministic value inde- pendently, it concatenates the value with the associated request and executes the three phase protocol to ensure that non-faulty replicas agree on a sequence number for the request and value. This prevents a faulty primary from causing replica state to diverge by sending different val- ues to different replicas. However, a faulty primary might send the same, incorrect, value to all replicas. Therefore, replicas must be able to decide deterministically whether the value is correct (and what to do if it is not) based only on the service state.

This protocol is adequate for most services (including NFS) but occasionally replicas must participate in selecting the value to satisfy a service’s speciﬁcation. This can be accomplished by adding an extra phase to the protocol: the primary obtains authenticated values proposed by the backups, concatenates 2 1 of them with the associated request, and starts the three phase protocol for the concatenated message. Replicas choose the value by a deterministic computation on the 2 1 values and their state, e.g., taking the median. The extra phase can be optimized away in the common case. For example, if replicas need a value that is “close enough” to that of their local clock, the extra phase can be avoided when their clocks are synchronized within some delta.
