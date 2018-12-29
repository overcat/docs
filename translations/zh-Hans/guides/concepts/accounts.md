---
title: 账户
---

账户是 Stellar 的核心数据结构。账户由公钥标识并保存在总账中。 总账中的其他所有东西，比如 offers(交易订单) 或 [trustlines(信任线)](./assets.md#trustlines)，都属于一个特定的账户。

帐户由 [Create Account](./list-of-operations.md#create-account) 操作创建。

帐户访问由公钥/密钥加密控制。如果你想使用一个账户提交一个事务（例如支付），那么你必须使用账户公钥对应密钥对事务进行签名。您还可以设置更复杂的[多重签名](./multi-sig.md)，以便对帐户提交的事务进行授权。


## 账户字段

账户拥有以下字段:

> #### Account ID (账户 ID)
> 最初用于创建帐户的公钥。您可以使用不同的密钥来对事务进行签名，但我们始终可以通过 Account ID 来识别该帐户。
>
> #### Balance (余额)
> 持有资产的总数，最小精度为 1/10,000,000。
>
> #### Sequence number (序列号)
> 帐户的当前事务序列号。这个数字的初始值等于创建账户时的总账号。
>
> #### Number of subentries (条目总数)
> 帐户拥有的[条目总数](./ledger.md#ledger-entries)。这个数字用来计算帐户的[最低余额](./fees.md#minimum-account-balance)。
>
> #### Inflation destination (通胀目标)
> (可选) 指定接受通货膨胀的帐户。每个帐户都可以投票将[通货膨胀](./inflation.md)发送到目的帐户。
>
> #### Flags (标识)
> 目前有三种标志可供[资产](./assets.md)发行者使用。
>
>   - **Authorization required (0x1)**: 其它账户必须在得到资产发行帐号的许可之后才能持有这项资产。
>   - **Authorization revocable (0x2)**: 允许资产发行账户撤销其他账户持有的这项资产。
>   - **Authorization immutable (0x4)**: 如果启用了这个标识，那么就再也不能设置任何授权标识，也永远不能删除帐户。
>
> #### Home domain (主域名)
> 你可以选择性的为账户添加一个主域名，客户端将会从该域名查找 [stellar.toml](./stellar-toml.md)。此域名应该为[绝对域名](https://en.wikipedia.org/wiki/Fully_qualified_domain_name)，例如 `example.com`。
>
> 通过联邦协议（The federation protocol），你可以使用主域名查找有关事务的 Memo 的更多信息，也可获取账户的[联邦地址(Stellar address)](https://www.stellar.org/developers/learn/concepts/federation.html#stellar-addresses)。有关联邦协议的更多信息，请参阅[联邦协议指南](./federation.md)。
>
> #### Thresholds (阈值)
> 操作具有不同的访问级别。此字段指定低访问级别、中访问级别和高访问级别的阈值以及主密钥的权重。更多信息请参阅[多重签名](./multi-sig.md)。
>
> #### Signers (签名账户)
> 用于[多重签名](./multi-sig.md)。 此字段会列出其他公钥及其权重，这些公钥可用于授权此帐户的交易。
>
> #### Liabilities (负债)
> 从协议版本 10 开始，帐户会记录它的负债情况，每个资产都有单独的负债记录。对某个资产来说，买入负债等于你的所有挂单中买入该资产的总和，卖出负债等于你的所有挂单中卖出该资产的总和。某项资产的卖出负债总应该小于或等于您拥有的此项资产的数量，而买入负债则应该小于或等于您可拥有的该资产的上限。