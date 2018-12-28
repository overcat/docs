---
title: 创建账户
---

在 Stellar 网络上您需要做的第一件事是创建一个账户。 您所有的资产都将存在于这个账户中，您可以通过该账户发送和接收资产——实际上，Stellar 网络中几乎所有的东西都和账户有关。

每个 Stellar 帐户都有一个**公钥**和一个**私密种子**。Stellar 通过使用公钥加密（也称非对称加密）来确保每个事务的安全性。公钥可以进行安全共享——其他人需要通过它来识别您的帐户并验证您是否授权了某笔事务。私密种子可以证明您拥有某个账户，所以建议您永远别与任何人分享私密种子。 举例来说，如果一个账户是一把锁的话，那么私密种子就是打开它的钥匙。同样的道理，任何知道您帐户的私密种子的人都可以控制您的帐户。

如果您熟悉公钥加密技术的话，可能会想知道私密种子与私钥有何不同。 私密种子实际上是为您的帐户生成公钥和私钥而服务的唯一秘密数据。 因此为了方便起见，Stellar 使用的是种子而不是私钥：要获得对帐户的完全控制权，只需提供私密种子而不需要同时提供公钥和私钥。[^1]

创建账户的第一步是创建自己的种子和密钥——因为种子需要保密，所以当您最终创建账户时，您只需将公钥发送到 Stellar 服务器。 您可以使用以下命令来生成种子和公钥：

<code-example name="生成密钥对">

```js
// 创建一个全新且独一无二的密钥对。
// 通过以下链接了解 KeyPair 对象：https://stellar.github.io/js-stellar-sdk/Keypair.html
var pair = StellarSdk.Keypair.random();

pair.secret();
// SAV76USXIJOBMEQXPANUOQM6F5LIOTLPDIDVRJBFFE2MDJXG24TAPUU7
pair.publicKey();
// GCFXHS4GXL6BVUCXBWXGTITROWLVYXQKQLF4YH5O5JT3YZXCYPAFBJZB
```

```java
// 创建一个全新且独一无二的密钥对。
// 通过以下链接了解 KeyPair 对象：https://stellar.github.io/java-stellar-sdk/org/stellar/sdk/KeyPair.html
import org.stellar.sdk.KeyPair;
KeyPair pair = KeyPair.random();

System.out.println(new String(pair.getSecretSeed()));
// SAV76USXIJOBMEQXPANUOQM6F5LIOTLPDIDVRJBFFE2MDJXG24TAPUU7
System.out.println(pair.getAccountId());
// GCFXHS4GXL6BVUCXBWXGTITROWLVYXQKQLF4YH5O5JT3YZXCYPAFBJZB
```

```go
package main

import (
	"log"

	"github.com/stellar/go/keypair"
)

func main() {
	pair, err := keypair.Random()
	if err != nil {
		log.Fatal(err)
	}

	log.Println(pair.Seed())
	// SAV76USXIJOBMEQXPANUOQM6F5LIOTLPDIDVRJBFFE2MDJXG24TAPUU7
	log.Println(pair.Address())
	// GCFXHS4GXL6BVUCXBWXGTITROWLVYXQKQLF4YH5O5JT3YZXCYPAFBJZB
}
```

</code-example>

现在有了种子和公钥，接下来就可以创建帐户了。 为了防止人们创建大量无用的帐户，每个帐户必须最少持有 1 Lumen(Lumen 是 Stellar 网络的内置货币)。[^2] 由于您还没有 Lumen，因此您无法支付创建账户所需的费用。 在现实世界中，您可以通过交易所购买一些 Lumen 来创建您的新帐户。[^3] 但在 Stellar 的测试网络中，你可以使用 Friendbot 为您创建账户。

要创建测试帐户的话，请将您创建的公钥发送给 Friendbot，它将使用公钥作为帐户 ID 创建一个新帐户并为其提供一笔资金。

<code-example name="创建一个测试账户">

```js
// 该 SDK 无法帮助您在测试网络中创建账户，所以你需要自己发起 HTTP 请求。
var request = require('request');
request.get({
  url: 'https://friendbot.stellar.org',
  qs: { addr: pair.publicKey() },
  json: true
}, function(error, response, body) {
  if (error || response.statusCode !== 200) {
    console.error('ERROR!', error || body);
  }
  else {
    console.log('SUCCESS! You have a new account :)\n', body);
  }
});
```

```java
// 该 SDK 无法帮助您在测试网络中创建账户，所以你需要自己发起 HTTP 请求。
import java.net.*;
import java.io.*;
import java.util.*;

String friendbotUrl = String.format(
  "https://friendbot.stellar.org/?addr=%s",
  pair.getAccountId());
InputStream response = new URL(friendbotUrl).openStream();
String body = new Scanner(response, "UTF-8").useDelimiter("\\A").next();
System.out.println("SUCCESS! You have a new account :)\n" + body);
```

```go
package main

import (
	"net/http"
	"io/ioutil"
	"log"
	"fmt"
)

func main() {
	// pair 是您之前创建的 keypair 实例
	address := pair.Address()
	resp, err := http.Get("https://friendbot.stellar.org/?addr=" + address)
	if err != nil {
		log.Fatal(err)
	}

	defer resp.Body.Close()
	body, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println(string(body))
}
```

</code-example>

现在是最后一步: 获取账户的详细信息并检查其余额。 账户可以有多个余额——每种货币各有一个余额。

<code-example name="获取账户详情">

```js
var server = new StellarSdk.Server('https://horizon-testnet.stellar.org');

// JS SDK 使用 Promise 来处理大部分操作，比如检索一个账户
server.loadAccount(pair.publicKey()).then(function(account) {
  console.log('Balances for account: ' + pair.publicKey());
  account.balances.forEach(function(balance) {
    console.log('Type:', balance.asset_type, ', Balance:', balance.balance);
  });
});
```

```java
import org.stellar.sdk.Server;
import org.stellar.sdk.responses.AccountResponse;

Server server = new Server("https://horizon-testnet.stellar.org");
AccountResponse account = server.accounts().account(pair);
System.out.println("Balances for account " + pair.getAccountId());
for (AccountResponse.Balance balance : account.getBalances()) {
  System.out.println(String.format(
    "Type: %s, Code: %s, Balance: %s",
    balance.getAssetType(),
    balance.getAssetCode(),
    balance.getBalance()));
}
```

```go
package main

import (
	"fmt"
	"log"

	"github.com/stellar/go/clients/horizon"
)

func main() {
	account, err := horizon.DefaultTestNetClient.LoadAccount(pair.Address())
	if err != nil {
		log.Fatal(err)
	}

	fmt.Println("Balances for account:", pair.Address())

	for _, balance := range account.Balances {
		log.Println(balance)
	}
}
```

</code-example>

现在您已经有了一个账户，您可以[开始收款与付款](transactions.md)。

<div class="sequence-navigation">
  <a class="button button--previous" href="index.html">上一页: Stellar 网络概览</a>
  <a class="button button--next" href="transactions.html">下一页: 收款与付款</a>
</div>



[^1]: 私钥仍用于加密数据和签署事务。 使用种子创建 `KeyPair`对象时，会立即生成私钥并在保存在程序内。

[^2]: Stellar 的一些其它特征，比如[信任线](../concepts/assets.md#trustlines), 需要更高的最低余额。 有关最低余额的更多信息，请参见[费用](../concepts/fees.md#minimum-account-balance)

[^3]: CoinMarketCap 维护了一份出售 Lumen 的交易所清单： http://coinmarketcap.com/currencies/stellar/#markets
