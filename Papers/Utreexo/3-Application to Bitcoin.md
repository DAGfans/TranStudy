> Source: https://eprint.iacr.org/2019/611.pdf
# 3 Application to Bitcoin
# 3 在比特币上的应用 

In contrast to account based systems where user balances can be incremented and decremented, the Bitcoin protocol keeps track of Unspent Transaction Outputs (referred to as UTXOs).
A transaction in Bitcoin has inputs and outputs (TXOs), the inputs referring to, and consuming, previously created outputs.
In this design, the only operations on the data set are create, read, and delete.
There is no modiﬁcation other than deletion possible of a set element after it is written.

与可以增加和减少用户余额的基于帐户的系统不同，比特币协议会跟踪未花费的交易输出（称为UTXO）。
比特币中的交易具有输入和输出（TXO），这些输入引用并使用先前创建的输出。
这种设计对数据集只能执行创建，读取和删除操作。
系统无法修改一个已经写入集合的元素，只能删除。

Cryptographic accumulators, ﬁrst introduced in [6], allow for set query operations without storing or revealing all members of the set.
Bitcoin’s UTXO set is well-suited to an accumulator construction; for each transaction we would like to query whether the TXOs being spent are indeed members of the UTXO set, and if not, reject the transaction.

最早在[6]中引入的密码学累加器允许执行集合查询操作，而无需存储或显示集合的所有成员。
比特币的UTXO集非常适合用来构造累加器；对于每笔交易，我们希望能查询被花费的TXO是否确实为UTXO集的成员，如果不是，则拒绝该交易。

In Bitcoin, clients verify all state changes.
This signiﬁcantly limits scalability of the system, as raising the resource requirements to participate reduces the number of participants in the system, in many cases forcing them to rely on SPV or custodial wallets.
As of early 2019, the initial synchronization process, called “Initial Block Download” (IBD), requires users to download over 200 gigabytes of historic data and to verify hundreds of millions of digital signatures.
The end state of the system is much smaller than the historic transcript – the UTXO set is closer to 4 gigabytes.

在比特币中，只要状态发生了变化，客户端就要进行验证。
这很大程度上限制了系统的可伸缩性，
因为这对系统参与者的资源【译注：比如存储容量、网速等】提出了更高的要求，
从而减少了系统参与者的数量。很多时候他们只能依赖SPV或托管钱包。
截至2019年初，称为“初始区块下载”（IBD）的初始同步过程要求用户下载超过200 GB的历史数据，并验证数亿个数字签名。
【译注：初始区块下载又叫初始同步。
英文原文是Initial Block Download（IBD），
是指一个全节点初次上线，在其有能力校验一个未确认的交易和最近新挖到的区块前，
必须下载并校验从创世区块起的整个交易历史记录的初始化过程】
但其实系统的最终状态比历史记录要小得多——UTXO集只有约4GB。

Long initial synchronization times and large data storage requirements present a burden to users and limit the reach and scalability of the system.
IBD times vary widely depending on hardware, and there are several different constraints which can be the limiting factor for IBD speed.
Disk I/O, especially the ability to rapidly perform random access reads, is very important for fast IBD.
In machines with solid state drives, disk I/O is not usually the limiting factor, and the CPU will be kept busy with signature veriﬁcation.
In machines using mechanical hard drives however, disk I/O is usually the limiting factor, and the CPU will spend most of its time waiting for disk reads to complete.
IBD times for a machine with an SSD can be around 6 hours, while the same machine using a mechanical hard drive can take more than a week.<sup>1</sup>

<sup>1</sup>Recently tested with Bitcoin Core 0.17.1 on a AMD Ryzen 1700 Machine using a Samsung 960 EVO 500GB SSD, in comparison with the same machine where the bitcoin folder was instead stored on a Samsung HD154UI 1.5TB mechanical drive.

较长的初始同步时间和大量的数据存储要求会给用户带来负担，并限制系统的覆盖范围和可伸缩性。
IBD时间因硬件而异，其速度受多个因素限制。
其中磁盘I/O，特别是快速执行随机访问读取的能力，对于快速IBD至关重要。
在带有固态驱动器的机器中，磁盘I/O通常不是限制因素，CPU将忙于签名验证。
但是，在使用机械硬盘的机器中，磁盘I/O通常就是限制因素，CPU将花费大部分时间等待磁盘读取完成。
具有SSD的计算机的IBD时间可能约为6个小时，而使用机械硬盘的同一台计算机则可能需要一周以上的时间。<sup>1</sup>

<sup>1</sup>这个测试是最近在使用三星960 EVO 500GB SSD的AMD Ryzen 1700机器上使用Bitcoin Core 0.17.1进行的，
比较对象是将比特币文件夹存储在三星HD154UI 1.5TB机械硬盘上的同一台机器。

Current Bitcoin client implementations store the UTXO set in an ondisk database, so that nodes can verify the existence and details of every UTXO when performing the read and delete operations at the end of the UTXO’s life in the database.
From the creation of a UTXO to just before it is spent, a duration that is often several years, the UTXO has no eﬀect on the system, and its database entry is never accessed.
A transaction in Bitcoin cannot query the state or existence of other UTXOs, and UTXOs are only read from disk when they are being spent.

当前比特币客户端的实现方式是将UTXO集存储在磁盘数据库中，使得节点在UTXO生命周期结束，执行读取和删除操作时，可以验证每个UTXO的存在状态和详细信息。
UTXO从其被创建到被花费的期间（往往为数年）内都不会对系统产生影响，并且其数据库条目永远不会被访问。
比特币中的交易无法查询交易外的其它UTXO的状态，也无法得知它们是否存在。
UTXO只有在被花费时才会从磁盘中被读取出来。

A design in which clients do not need to store UTXOs during this dormant period oﬀers many beneﬁts.
Currently the system’s state can be stored on inexpensive hardware, but as there is is no limit on the state’s size, this may not continue to be the case.
(Bitcoin’s block size limit does limit the growth rate of the UTXO set to approximately 1MB every 10 minutes, but does not impose an absolute limit on the set size).
Omitting dormant UTXOs not only helps long term scalability, but also increases synchronization speed as disk reads and writes are minimized.
Full validation is possible using only data arriving over the network and a small in-memory state representation.
Additionally, this type of client allows for better code separation as evaluating the validity of a transaction or block can be done in isolation, since all data needed to validate a transaction is comparable in size to the transaction itself, rather than the size of the entire UTXO database as is currently the case.

如果客户端在UTXO的休眠期中无需存储UTXO，那将带来很多好处。
按当前的设计，虽然我们可以将系统状态存储在便宜的硬件上，但是由于状态大小没有限制，因此这样的设计不可持续。
（虽然比特币的块大小限制确实将UTXO的增长率控制在了每10分钟1MB，但却并没有限制整个UTXO集合的大小）。
删掉休眠的UTXO不仅有助于长期扩容，而且还可以最大程度地减少磁盘读写，从而提高同步速度。
相应地，我们可以仅通过网络数据和较小的内存状态来做完全验证。
此外，在这类客户端上，交易或区块的有效性验证工作可以彼此隔离，从而实现更好的代码分离，
因为验证交易所需的所有数据的大小可以缩小到与交易本身相当，
而不是跟现在一样，验证一笔交易要用到的数据就跟整个UTXO数据库一样大。

# Reference
[6] Josh Benaloh and Michael de Mare. One-way accumulators: A decentralized alternative to digital sinatures (extended abstract). In EUROCRYPT, 1993.
