---
title: 总账
---

**总账**代表 Stellar 网络在给定时间点的状态。它包含所有帐户和余额的列表、分布式交易所中的所有订单和任何其它持久存在的数据。

网络历史中的第一个总账被称为创世总账。

网络会通过 [Stellar 共识协议(SCP)](https://www.stellar.org/developers/learn/concepts/scp.html)，就应该将哪个[事务集](./transactions.md#transaction-set)应用在最近关闭的总账中达成共识。当一个新的事务集被应用后，一个新的“最近关闭的总账”随之产生。

每个总账依靠加密算法连接着前一个独一无二的总账，由此形成了一个可以一直追溯到创世总账的总账链。

我们递归地定义总账的序列号：
* 创世总账的序列号为 1。
* 序号列为 n 的总账的后一个总账的序列号为 n+1。

## 总账头部
每个总账都有一个**总账头部**。这个头部包含了对当前总账中实际数据的引用以及对前一个总账的引用。这里的引用是所引用内容的加密 hash —— 这些 hash 值的类似于数据结构中的指针，但它具有附加的安全保证。

你可以把历史总账链看作是总账账头的链接列表：

[创世总账] <—— [总账头 1] <----- ... <—— [总账头 n]

有关对象佛如定义，请参见协议文件。
[`src/xdr/Stellar-ledger.x`](https://github.com/stellar/stellar-core/blob/master/src/xdr/Stellar-ledger.x)

每个总账头部都有以下字段：

- **Version**: 这个总账使用的协议版本号。
- **Previous Ledger Hash**: **上一个总账**的 Hash，总账之间通过该值相连，你可以通过该值追溯到创世总账。
- **SCP value**: 网络中的所有节点都在运行 SCP，并就特定的值达成一致。这个值存储在这里以及以下三个字段中。
  - **Transaction set hash**: 应用于上一个总账的事务集的 hash。

  - **Close time**: 当前总账关闭的时间。格式为 UNIX 时间戳。

  - **Upgrades**: 网络如何调整[基本费用](./fees.md)并升级到新的协议版本。该值一般为空。想要了解更多，请阅读这篇文章——[版本](./versioning.md)。
- **Transaction set result hash**: 最终应用的事务集的 Hash。严格地说，这些数据并不是验证事务结果所必需的。但是，此数据使实体可以更轻松地验证给定事务的结果，而不必将事务集应用于上一个总帐。
- **Bucket list hash**: 此分类帐中所有对象的 Hash。包含所有对象的数据结构称为[存储桶列表](https://github.com/stellar/stellar-core/tree/master/src/bucket)。
- **Ledger sequence**: 该总账的序列号。
- **Total coins**: 现存 Lumens 总数。
- **Fee pool**: 作为费用支付的 Lumens 数量。这些 Lumens 将被添加到通货膨胀池中，并在下次通货膨胀运行时重置为 0。

- **Inflation sequence**: 通货膨胀运行的次数。
- **ID pool**: 上次使用的全局 id。这些 id 用于生成对象。
- **Maximum Number of Transactions**: 对于给定的总账，验证节点最多能处理多少[事务](./transactions.md)。如果提交的事务数量超过该值, 验证节点将优先处理手续费高的事务。
- **Base fee**: [事务](./transactions.md)中每个[操作](./operations.md)需要支付的[手续费](./fees.md#transaction-fee)。该值以 stroops(1 stroop = 1/10,000,000 Lumen) 为单位。
- **Base reserve**: 网络当前的[基本储备金](./fees.md#minimum-account-balance)，它可以用于计算最低账户余额。
- **Skip list**: 之前的总账的 Hash。你可以通过该列表快速的跳转到之前的总账，而不用一个个查找过去。此列表中有 4 个总账的 hash。这 4 个总账的序列号发布为 skipList[0] mod(50)，skipList[1] mod(5000)，skipList[2] mod(50000)，skipList[3] mod(500000)。

# 总账条目

总账是**条目**的集合。目前有 4 种类型的总账条目，它们被定义在 [`src/xdr/Stellar-ledger-entries.x`](https://github.com/stellar/stellar-core/blob/master/src/xdr/Stellar-ledger-entries.x) 中。

## 账户条目
此条目代表一个[帐户](https://stellar-docs.overcat.me/guides/concepts/accounts.html)。在 Stellar 中，一切都围绕帐户构建的：事务由帐户执行，帐户控制着对余额的访问权限。

其他条目是账户条目的子条目。帐户每添加一个新的子条目都会增加帐户需要持有的最低账户余额。想要了解更多请参阅[费用和最低账户余额](https://stellar-docs.overcat.me/guides/concepts/fees.html#minimum-account-balance)。

## 信任线条目
用户需要通过创建一条[信任线](./assets.md)来信任特定发行账户发行的特定资产。

信任线条目定义了使用这种资产的规则。该规则可以由用户定义——例如，设置可持有该资产的最大数量以限制风险，也可以有发行账户定义——例如，启用授权标识。

## 订单条目
订单是帐户在订单簿中创建的条目。它们是在 Stellar 网络内自动进行简单交易的一种方式。有关订单的更多信息，请参阅[分布式交易所](https://stellar-docs.overcat.me/guides/concepts/exchange.html)。

## 数据条目
数据条目是附加到帐户上的键值对。系统允许帐户持有者将任意数据附加到其帐户。你可以灵活的将用于特定应用程序的数据项添加到总帐中。
