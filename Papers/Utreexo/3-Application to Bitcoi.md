> Source: https://eprint.iacr.org/2019/611.pdf
# 3 Application to Bitcoin
# 3 比特币的应用 

In contrast to account based systems where user balances can be incremented and decremented, the Bitcoin protocol keeps track of Unspent Transaction Outputs (referred to as UTXOs).
A transaction in Bitcoin has inputs and outputs (TXOs), the inputs referring to, and consuming, previously created outputs.
In this design, the only operations on the data set are create, read, and delete.
There is no modiﬁcation other than deletion possible of a set element after it is written.

与可以增加和减少用户余额的基于帐户的系统相比，比特币协议跟踪未花费的交易输出（称为UTXO）。
比特币中的交易具有输入和输出（TXO），这些输入引用并使用先前创建的输出。
在这种设计中，对数据集的唯一操作是创建，读取和删除。
除了在写入集合元素后可以删除之外，没有其他修改方式。

Cryptographic accumulators, ﬁrst introduced in [6], allow for set query operations without storing or revealing all members of the set.
Bitcoin’s UTXO set is well-suited to an accumulator construction; for each transaction we would like to query whether the TXOs being spent are indeed members of the UTXO set, and if not, reject the transaction.

最早在[6]中引入的加密累加器允许进行集合查询操作，而无需存储或显示集合的所有成员。
比特币的UTXO集非常适合于累加器构造； 对于每笔交易，我们想查询被花费的TXO是否确实是UTXO集的成员，如果不是，则拒绝该交易。

In Bitcoin, clients verify all state changes.
This signiﬁcantly limits scalability of the system, as raising the resource requirements to participate reduces the number of participants in the system, in many cases forcing them to rely on SPV or custodial wallets.
As of early 2019, the initial synchronization process, called “Initial Block Download” (IBD), requires users to download over 200 gigabytes of historic data and to verify hundreds of millions of digital signatures.
The end state of the system is much smaller than the historic transcript – the UTXO set is closer to 4 gigabytes.

在比特币中，客户端验证所有状态更改。
这显着限制了系统的可伸缩性，因为提高参与的资源需求会减少系统中的参与者数量，在许多情况下，迫使他们依赖SPV或托管钱包。
截至2019年初，称为“初始块下载”（IBD）的初始同步过程要求用户下载超过200 GB的历史数据，并验证数亿个数字签名。
系统的最终状态比历史记录要小得多– UTXO集只有约 4GB。

Long initial synchronization times and large data storage requirements present a burden to users and limit the reach and scalability of the system.
IBD times vary widely depending on hardware, and there are several diﬀerent constraints which can be the limiting factor for IBD speed.
Disk I/O, especially the ability to rapidly perform random access reads, is very important for fast IBD.
In machines with solid state drives, disk I/O is not usually the limiting factor, and the CPU will be kept busy with signature veriﬁcation.
In machines using mechanical hard drives however, disk I/O is usually the limiting factor, and the CPU will spend most of its time waiting for disk reads to complete.
IBD times for a machine with an SSD can be around 6 hours, while the same machine using a mechanical hard drive can take more than a week.<sup>1</sup>

<sup>1</sup>Recently tested with Bitcoin Core 0.17.1 on a AMD Ryzen 1700 Machine using a Samsung 960 EVO 500GB SSD, in comparison with the same machine where the bitcoin folder was instead stored on a Samsung HD154UI 1.5TB mechanical drive.

较长的初始同步时间和大量的数据存储要求给用户带来负担，并限制了系统的覆盖范围和可伸缩性。
IBD时间因硬件而异，并且有几个不同的约束条件可能成为IBD速度的限制因素。
磁盘I/O，特别是快速执行随机访问读取的能力，对于快速IBD至关重要。
在带有固态驱动器的机器中，磁盘I/O通常不是限制因素，CPU将忙于签名验证。
但是，在使用机械硬盘的机器中，磁盘I/O通常是限制因素，CPU将花费大部分时间等待磁盘读取完成。
具有SSD的计算机的IBD时间可能约为6个小时，而使用机械硬盘的同一台计算机则可能需要一周以上的时间。<sup>1</sup>

<sup>1</sup>最近在使用三星960 EVO 500GB SSD的AMD Ryzen 1700机器上使用Bitcoin Core 0.17.1进行了测试，与将比特币文件夹存储在三星HD154UI 1.5TB机械硬盘上的同一台机器相比。

Current Bitcoin client implementations store the UTXO set in an ondisk database, so that nodes can verify the existence and details of every UTXO when performing the read and delete operations at the end of the UTXO’s life in the database.
From the creation of a UTXO to just before it is spent, a duration that is often several years, the UTXO has no eﬀect on the system, and its database entry is never accessed.
A transaction in Bitcoin cannot query the state or existence of other UTXOs, and UTXOs are only read from disk when they are being spent.

当前的比特币客户端实现将UTXO集存储在磁盘数据库中，以便节点在UTXO生命周期结束时执行读取和删除操作时，可以验证每个UTXO的存在状态和详细信息。
从创建UTXO到使用它的时间（经常为数年），UTXO不会对系统产生影响，并且其数据库条目永远不会被访问。
比特币中的交易无法查询其他UTXO的状态或存在状态，并且仅在使用UTXO时才从磁盘读取它们。

A design in which clients do not need to store UTXOs during this dormant period oﬀers many beneﬁts.
Currently the system’s state can be stored on inexpensive hardware, but as there is is no limit on the state’s size, this may not continue to be the case.
(Bitcoin’s block size limit does limit the growth rate of the UTXO set to approximately 1MB every 10 minutes, but does not impose an absolute limit on the set size).
Omitting dormant UTXOs not only helps long term scalability, but also increases synchronization speed as disk reads and writes are minimized.
Full validation is possible using only data arriving over the network and a small in-memory state representation.
Additionally, this type of client allows for better code separation as evaluating the validity of a transaction or block can be done in isolation, since all data needed to validate a transaction is comparable in size to the transaction itself, rather than the size of the entire UTXO database as is currently the case.

在这种休眠期中，客户无需存储UTXO的设计有很多好处。
目前，系统状态可以存储在便宜的硬件上，但是由于状态大小没有限制，因此这样的设计不可持续。
（比特币的块大小限制确实将UTXO设置的增长率每10分钟限制为大约1MB，但没有限制集合的大小上限）。
裁剪休眠的UTXO不仅有助于长期扩容，而且还可以最大程度地减少磁盘读写，从而提高同步速度。
仅使用通过网络访问的数据和较小的内存状态表示格式，即可进行完全验证。
此外，这种类型的客户端可以实现更好的代码分离，因为可以单独评估交易或区块的有效性，因为验证交易所需的所有数据的大小都与交易本身相当，而非当前的情况那样是整个交易的UTXO数据库的大小。

# Reference
[6] Josh Benaloh and Michael de Mare. One-way accumulators: A decentralized alternative to digital sinatures (extended abstract). In EUROCRYPT, 1993.