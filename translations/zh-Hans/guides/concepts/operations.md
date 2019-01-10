---
title: 操作
---

[事务](./transactions.md)中包含了[一系列的操作](./list-of-operations.md)，每个操作都会改变总账。

以下是目前可用的操作类型：
- [Create Account](./list-of-operations.md#create-account)
- [Payment](./list-of-operations.md#payment)
- [Path Payment](./list-of-operations.md#path-payment)
- [Manage Offer](./list-of-operations.md#manage-offer)
- [Create Passive Offer](./list-of-operations.md#create-passive-offer)
- [Set Options](./list-of-operations.md#set-options)
- [Change Trust](./list-of-operations.md#change-trust)
- [Allow Trust](./list-of-operations.md#allow-trust)
- [Account Merge](./list-of-operations.md#account-merge)
- [Inflation](./list-of-operations.md#inflation)
- [Manage Data](./list-of-operations.md#manage-data)
- [Bump Sequence](./list-of-operations.md#bump-sequence)

除非你在操作中明确指定了发起操作的源账户，否则默认将发起事务的源账户视为该操作的源账户。

## 阈值(Thresholds)

每个操作都属于一个特定的阈值类别: 低、中或高。阈值定义了成功执行操作所需的权限级别。

* 低等权限操作：
  * AllowTrustTx
    * 允许或冻结其它账户持有的本账户发行的资产。
  * BumpSequence
* 中等权限操作：
  * 除高低等权限操作之外的所有操作。
* 高等权限操作：
  * AccountMerge
    * 将一个账户合并到另外一个账户中
  * 使用 SetOptions 操作设置签名账户和阈值
    * 用于更改签名账户集和阈值


## 验证操作是否有效

在[事务生命周期](./transactions.md#life-cycle)中，操作可能在其中两个阶段失败。首先是将事务提交给网络时，提交事务的节点会检查操作的有效性：在**有效性检查**中，节点进行一些粗略的检查，以确保事务所需的信息是完整的，然后将其包含在事务集中，并将事务转发给网络中的其它节点。

有效性检查仅查看源帐户的状态。它确保：
1) 这个事务具有足够的帐户签名，以满足该操作所需的阈值。
2) 这一步不会检查事务的其它参数是否合理，而是在事务被应用到总账是执行其它更多的检查，比如余额是否足够。

一旦事务通过了第一次有效性检查，它就会被打包到事务集中随后广播到网络中。事务作为事务集的一部分，会被打包到总帐中。此时，无论事务成功还是失败，都会从源帐户中收取手续费。稍后，系统将按照事务发生的顺序验证它们的序列号和签名。如果一个事务中有任何操作失败，则整个事务都会失败，账户将不会发生变动（但是手续费会扣除）。

## 结果

对于每个操作，都有一个与之匹配的结果类型。在成功的情况下，这个结果使用户知道这个操作产生了怎样的影响，在失败的情况下，它会向用户提供详细的错误信息。

Stellar Core 按顺序将结果存放在 txhistory 表中以供其它组件使用。例如，Stellar Core 中的历史模块将对 txhistory 表中的数据进行进一步处理，以让它得以永久保存。它也可以用于外部程序，比如 Horizon 会从该表中收集必要的信息。

## 涉及多个帐户的事务

通常情况下，事务只涉及单一帐户上的操作。例如，如果帐户 a 想发送 Lumens 到帐户 b，这个事务只要获取到帐户 a 的授权即可。

但是，我们也可以发起一个由多个帐户作为源账户的操作的事务。在这种情况下，要完成对操作进行授权，事务信封必须包含所涉及的每个帐户的签名。例如，一个事务中包含了两个支付操作，账户 a 和账户 b 分别发送一笔资产给账户 c，那么这个事务在提交到网络之前需要得到账户 a 和账户 b 的授权。


## 例子
### 1. 不依赖第三方进行资产交换

  Anush 想给 Bridget 发送一些 XLM (操作 1) 用于兑换一些 BTC (操作 2)。

  事务详情如下：
  * 源帐号 = `Anush_account`
  * 操作 1
    * 源帐号  = _null_
    * 发送 XLM --> `Bridget_account`
  * 操作 2
    * 源帐号 = _`Bridget_account`
    * 发送 BTC --> `Anush_account`

   所需的签名：
  * 操作 1: 需要 `Anush_account` 的签名(该操作默认使用事务的源帐户)（中等阈值）。
  * 操作 2: 需要 `Bridget_account` 的签名（中等阈值）。
  * 这个事务需要 `Anush_account` 的签名以验证该事务是由 `Anush_account` 发起的（低等阈值）。

因此，如果 `Anush_account` 和 `Bridget_account` 都签署了该事务，它才是有效的。

### 2. 工作机

   锚点希望每台机器都能单独的对基础账户（在这里指包含了大量资产的账户）进行操作。这样的话，每台机器将使用其本地帐户提交事务，并使用自己独立的序列号，而不用依赖于基础账户。想要了解交易序列号，请参阅[关于事务的文档](./transactions.md)。

   * 每台机器都有一个与之对应的密钥对。我们假设这里只有三台机器：Machine_1, Machine_2, 和 Machine_3。(实际上，锚点可以设置任意多的机器。)
   * 将这三台机器的公钥添加到基础账户的签名列表中，并赋予它们中等签名权限，这样工作机就可以代表基础账户进行签名了。(想要了解更多与签名相关的知识，请查阅[关于多重签名的文档](multi-sig.md)。)
   * 当一台机器（比如 Machine_2）想要向网络提交事务时，它会这样构建事务：
      * 源账户=_Machine_2 的公钥_
      * 序列号=_Machine_2 的账户的序列号_
      * 操作
        * 源账户=_基础账户(baseAccount)_
        * 发送资产 --> 目标账户
   * 使用 Machine_2 的密钥进行签名。

   回想一下[事务文档](transactions.md)中的描述，所有事务都有自己的特定序列号，且这个序列号需要与源账户的序列号匹配。这种方案的优点是每台机器都可以增加其序列号并提交事务，而不会使其他机器提交的事务失效。通过为多个工作机器配置多个账户，此锚点能够提交尽可能多的事务而不会发生序列号冲突。

### 3. 耗时较长的事务

需要多方签署的事务，可能需要花费较长的时间才会签署并提交到网络中，如例 1 中 Anush 和 Bridget 之间的交易。因为所有的交易都是用特定的序列号构造的，等待签名并提交的这段时间，Anush 的帐户无法再进行其它操作，否则会导致他和 Bridget 的交易失败。为了避免这种情况，可以使用类似于示例 2 的方案。

  Anush 创建一个临时账户 `Anush_temp`, 并给它发送一些 XLM 以激活它，并将 `Anush_account` 的公钥作为签署者添加到 `Anush_temp` 的签名列表中，权重需要高于低等阈值。

  事务详情如下：
  * 源账户=_Anush_temp_
  * 序列号=_Anush_temp 的序列号_
  * 操作 1
    * 源账户=_Anush_account_
    * 发送 XLM -> Bridget_account
  * 操作 2
    * 源账户=_Bridget_account_
    * 发送 BTC -> Anush_account

  这个事务需要被 Anush_account 和 Bridget_account 签名, 但是它的序列号由 Anush_temp 提供。

  如果 `Anush_account` 想回收 `Anush_temp` 账户中的 XLM，则可以在事务中添加“操作 3”。如果你想这样做，`Anush_account` 必须将 `Anush_temp` 添加到签名列表中，并赋予它高等签名权限：
  * 操作 3
    * source=_null_
    * 账户合并 -> "Anush_account"
