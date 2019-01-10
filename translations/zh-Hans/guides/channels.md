---
title: 信道 
---
*向网络高速提交事务的方法*

如果你以较高速率或不同进程向网络提交[事务](./concepts/transactions.md)时，必须要小心让事务使用正确的序列号(sequence)次序进行提交。因为通常情况下你会使用Horizon进行提交事务，但其并不能保证给定的事务会被[Stellar core](https://github.com/stellar/stellar-core)接收到，除非关账成功。这可能会导致问题出现，也就是说事务可能乱序到达stellar core并引发不正确的序列号错误。如果你等待关账以避免类似错误，那么将会大大降低事务提交的速率。

避免此类问题的方式是使用**信道**的概念。

信道即使用另外一个恒星账户作为事务的“源”账户。注意恒星中的每个事务都要源账户，可以与事务中被操作改变的账户不同。事务的源账户会支付事务手续费以及消耗序列号。你可以在事务中使用某一账号来进行支付[操作](./concepts/operations.md)。尽管资金是由该账户发出，构建这些事务的信道账户将会消耗掉它们的序列号。

信道使用了事务的源账户可以与事务内操作的源账户不同这一特性。利用这一点你可以按需创建多个信道来维系你希望的事务速率。

当然，你需要使用信道账户和实际账户的密钥来签署该事务。

例如：
```
StellarSdk.Network.useTestNetwork();
// channelAccounts[] 是信道账户数组，每一个代表一个信道
// channelKeys[] 是信道账户的密钥数组
// channelIndex 是你想要发生事务的信道

// 创建从baseAccount到customerAddress的支付
var transaction =
  new StellarSdk.TransactionBuilder(channelAccounts[channelIndex])
    .addOperation(StellarSdk.Operation.payment({
      source: baseAccount.address(),
      destination: customerAddress,
      asset: StellarSdk.Asset.native(),
      amount: amountToSend
    }))
    .build();

  transaction.sign(baseAccountKey);   // base account需要签名验证支付
  transaction.sign(channelKeys[channelIndex]);  // 信道需要签名验证源账户
``` 
