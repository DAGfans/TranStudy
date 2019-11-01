> Source: https://eprint.iacr.org/2019/611.pdf
# 3 Application to Bitcoin
# 3 比特币的应用 

In contrast to account based systems where user balances can be incremented and decremented, the Bitcoin protocol keeps track of Unspent Transaction Outputs (referred to as UTXOs).
A transaction in Bitcoin has inputs and outputs (TXOs), the inputs referring to, and consuming, previously created outputs.
In this design, the only operations on the data set are create, read, and delete.
There is no modiﬁcation other than deletion possible of a set element after it is written.

Cryptographic accumulators, ﬁrst introduced in [6], allow for set query operations without storing or revealing all members of the set.
Bitcoin’s UTXO set is well-suited to an accumulator construction; for each transaction we would like to query whether the TXOs being spent are indeed members of the UTXO set, and if not, reject the transaction.

In Bitcoin, clients verify all state changes.
This signiﬁcantly limits scalability of the system, as raising the resource requirements to participate reduces the number of participants in the system, in many cases forcing them to rely on SPV or custodial wallets.
As of early 2019, the initial synchronization process, called “Initial Block Download” (IBD), requires users to download over 200 gigabytes of historic data and to verify hundreds of millions of digital signatures.
The end state of the system is much smaller than the historic transcript – the UTXO set is closer to 4 gigabytes.

Long initial synchronization times and large data storage requirements present a burden to users and limit the reach and scalability of the system.
IBD times vary widely depending on hardware, and there are several diﬀerent constraints which can be the limiting factor for IBD speed.
Disk I/O, especially the ability to rapidly perform random access reads, is very important for fast IBD.
In machines with solid state drives, disk I/O is not usually the limiting factor, and the CPU will be kept busy with signature veriﬁcation.
In machines using mechanical hard drives however, disk I/O is usually the limiting factor, and the CPU will spend most of its time waiting for disk reads to complete.
IBD times for a machine with an SSD can be around 6 hours, while the same machine using a mechanical hard drive can take more than a week.<sup>1</sup>

<sup>1</sup>Recently tested with Bitcoin Core 0.17.1 on a AMD Ryzen 1700 Machine using a Samsung 960 EVO 500GB SSD, in comparison with the same machine where the bitcoin folder was instead stored on a Samsung HD154UI 1.5TB mechanical drive.

Current Bitcoin client implementations store the UTXO set in an ondisk database, so that nodes can verify the existence and details of every UTXO when performing the read and delete operations at the end of the UTXO’s life in the database.
From the creation of a UTXO to just before it is spent, a duration that is often several years, the UTXO has no eﬀect on the system, and its database entry is never accessed.
A transaction in Bitcoin cannot query the state or existence of other UTXOs, and UTXOs are only read from disk when they are being spent.

A design in which clients do not need to store UTXOs during this dormant period oﬀers many beneﬁts.
Currently the system’s state can be stored on inexpensive hardware, but as there is is no limit on the state’s size, this may not continue to be the case.
(Bitcoin’s block size limit does limit the growth rate of the UTXO set to approximately 1MB every 10 minutes, but does not impose an absolute limit on the set size).
Omitting dormant UTXOs not only helps long term scalability, but also increases synchronization speed as disk reads and writes are minimized.
Full validation is possible using only data arriving over the network and a small in-memory state representation.
Additionally, this type of client allows for better code separation as evaluating the validity of a transaction or block can be done in isolation, since all data needed to validate a transaction is comparable in size to the transaction itself, rather than the size of the entire UTXO database as is currently the case.