# Byteball: A Decentralized System for Storage and Transfer of Value
# 字节雪球: 去中心化的存储和价值转移系统

#### Anton Churyumov 
#### tonych@byteball.org

## Abstract 
## 摘要 

Byteball is a decentralized system that allows tamper proof storage of arbitrary data, including data that represents transferrable value such as currencies, property titles, debt, shares, etc. 
Storage units are linked to each other such that each storage unit includes one or more hashes of earlier storage units, which serves both to confirm earlier units and establish their partial order. 
The set of links among units forms a DAG (directed acyclic graph). 
There is no single central entity that manages or coordinates admission of new units into the database, everyone is allowed to add a new unit provided that he signs it and pays a fee equal to the size of added data in bytes. 
The fee is collected by other users who later confirm the newly added unit by including its hash within their own units. 
As new units are added, each earlier unit receives more and more confirmations by later units that include its hash, directly or indirectly.

Byteball是一个去中心化的系统，允许防篡改地存储任意数据，包括表示货币，财产所有权，债务，股份等可转移价值的数据。
存储单元彼此链接，以使每个存储单元包含一个或多个先前存储单元的散列，这既用于确认先前的单元，也用于确定它们的偏序。 (译注: 偏序这里指两个单元之间的顺序, 但是无法确定这些单元在在所有单元之间的顺序)
单元之间的链接集形成了一个DAG（有向无环图）。 
没有一个单一的中央实体管理或协调新单元加入到数据库中，每个节点都可以添加一个新的单元，只要它签名并支付与添加数据大小相等的费用（以字节为单位）。 
该费用由其他用户收集，后者通过将其散列包含在其自己的单元中来确认新添加的单元。 
随着新单元的添加，每个较早的单元将通过后续单元（包括其散列）直接或间接地收到越来越多的确认。

There is an internal currency called ‘bytes’ that is used to pay for adding data into the decentralized database. 
Other currencies (assets) can also be freely issued by anyone to represent property rights, debt, shares, etc. 
Users can send both bytes and other currencies to each other to pay for goods/services or to exchange one currency for another; 
the transactions that move the value are added to the database as storage units. 
If two transactions try to spend the same output (double-spend) and there is no partial order between them, both are allowed into the database but only the one that comes earlier in the total order is deemed valid. 
Total order is established by selecting a single chain on the DAG (the main chain) that is attracted to units signed by known users called witnesses. 
A unit whose hash is included earlier on the main chain is deemed earlier on the total order. 
Users choose the witnesses by naming the user-trusted witnesses in every storage unit. 
Witnesses are reputable users with real-world identities, and users who name them expect them to never try to double-spend. 
As long as the majority of witnesses behave as expected, all double-spend attempts are detected in time and marked as such. 
As witnesses-authored units accumulate after a user’s unit, there are deterministic (not probabilistic) criteria when the total order of the user’s unit is considered final.

有一种称为“字节(bytes)”的内部货币，用于支付将数据添加到去中心化数据库的费用。
其他货币（资产）也可以由任何人免费发行以代表产权，债务，股票等。
用户可以相互将字节和其他货币发送给对方以支付商品/服务或兑换一种货币为另一种货币;
转移价值的交易将作为存储单元添加到数据库中。
如果两个交易尝试花费相同的输出（双花）并且它们之间没有偏序，则两个交易都被允许进入数据库，但只有全序中较早的交易才被视为有效。(译注:  全序代表所有交易都需要排序, 而上文中的偏序只要指定的交易集可以排序)
全序是通过选择被称为见证人的已知用户签名的单元组成的DAG上的单条链（主链）来建立的。
单元的散列如果在主链上较早地被包含则在全序中被认为较早。
用户通过在每个存储单元中指定信任的见证人来选择它们。
见证人是有真实世界身份的且有信誉的用户，并且指定它们的用户也期望它们永远不会尝试双花。
只要大多数见证人的行为如预期一样，所有的双花尝试都会被及时发现并被标记。
随着见证人产生的单元在用户单元之后积累，会有明确的（非概率性）标准来判断用户单元的全序能否最终确认。

Users store their funds on addresses that may require more than one signature to spend (multisig). 
Spending may also require other conditions to be met, including conditions that are evaluated by looking for specific data posted to the database by other users (oracles).

用户可将他们的资金存储在需要多重签名（multisig）才能花费的地址上。 
花费也可能需要满足其他条件，例如需要通过查找其他用户发布到数据库的特定数据（预言机）才能得到。

Users can issue new assets and define rules that govern their transferability. 
The rules can include spending restrictions such as a requirement for each transfer to be cosigned by the issuer of the asset, which is one way for financial institutions to comply with existing regulations. 
Users can also issue assets whose transfers are not published to the database, and therefore not visible to third parties. 
Instead, the information about the transfer is exchanged privately between users, and only a hash of the transaction and a spend proof (to prevent double-spends) are published to the database.

用户可以发布新资产并定义规则来管理转账。 
这些规则可以包括花费限制，例如每次转账要求由资产发行人联合签名，这是金融机构遵守现有规定的一种方式。 
用户也可以发布不公布转账信息到数据库的资产，因此对第三方不可见。 
相反，有关转账的信息是在用户之间私下交换的，只有交易散列和花费证明（防止双花）才会公布到数据库。
