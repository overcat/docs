---
title: 联邦服务 
sequence:
  previous: 2-bridge-server.md
  next: 4-compliance-server.md
---

在测试桥接服务时，我们在事务中添加了`memo`，以便识别要入账的客户账户。 但是使用恒星的其他人和组织可能不知道需要这样添加。他们要如何知晓呢？

[恒星联邦协议](../concepts/federation.md)允许您将人类可读的地址如`amy * your_org.com`[^friendly_names]转换为账户ID。它还包含在事务的`memo`中应包含的内容。发起付款时，您需要首先联系联邦服务以确定要支付到的恒星账户ID。 幸运的是，桥接服务会为您完成此操作。

![支付流程图示](assets/anchor-send-payment-federation.png)

Stellar.org 提供了一个[预编译的联邦服务](https://github.com/stellar/go/tree/master/services/federation)可以链接到现有用户数据库，但您也可以编写自己的服务。

## 创建数据库

恒星联邦服务设计为可以连接到任何您现有的数据库，只要其中拥有系列账户名即可。实际上，它会将联邦查询请求转换为SQL查询。它目前支持MySQL，PostgreSQL和SQLite3。

您的数据库至少应该有一个表，其中有一列用于标识每个账户记录的名称。[^federation_tables]在现有系统中，您可能有一个名为`accounts`的表，如下所示：

| id | first_name | last_name | friendly_id         |
|----|------------|-----------|---------------------|
| 1  | Tunde      | Adebayo   | tunde_adebayo       |
| 2  | Amy        | Smith     | amy_smith           |
| 3  | Jack       | Brown     | jack_brown          |
| 4  | Steintór   | Jákupsson | steintor_jakupsson  |
| 5  | Sneha      | Kapoor    | sneha_kapoor        |

其中 Tunde的恒星地址应该为 `tunde_adebayo*your_org.com`。


## 下载并配置联邦服务

接下来，按操作系统平台[下载最新版联邦服务](https://github.com/stellar/go/releases)。将其安装到任意地方，并在同一目录内创建一个名叫 `federation.cfg`的文件。该文件用于存储配置。如下所示：

<code-example name="federation.cfg">

```toml
port = 8002

[database]
type = "mysql" # Or "postgres" or "sqlite3"
dsn = "dbuser:dbpassword@/internal_accounts"

[queries]
federation = "SELECT 'GAIGZHHWK3REZQPLQX5DNUN4A32CSEONTU6CMDBO7GDWLPSXZDSYA4BU' as id, friendly_id as memo, 'text' as memo_type FROM accounts WHERE friendly_id = ? AND ? = 'your_org.com'"
reverse-federation = "SELECT friendly_id, '' as domain FROM accounts WHERE ? = ''"

# 联邦服务必须配置HTTPS，在此指定SSL证书及密钥文件。
# 如果联邦服务在代理或负载均衡之后，此项可忽略
[tls]
certificate-file = "server.crt"
private-key-file = "server.key"
```

</code-example>

确保使用正确的凭据和数据库名称。还要更新`federation`查询中`domain`值以使用您的实际域名而不是`your_org.com`。

`federation` 部分是一个SQL查询，给出恒星地址的两部分作为参数，如 `tunde_adebayo*your_org.com`分解为`tunde_adeboyo`和`your_org.com`，返回`id`，`memo`，和`memo_type`列。

由于我们将所有地址映射到我们的基本账户，因此我们始终为`id`返回基本账户ID。 与第一部分一致，我们希望将账户的`friendly_id`作为文本memo。

`reverse-federation` 部分是必需的。但是由于我们将所有客户账户都映射到同一个恒星账户，我们需要确保该查询总是返回空行。

现在可以运行服务了！（不像桥接服务，无需迁移任何数据库。）

```bash
./federation
```


## 更新Stellar.toml

最后，其他人必须知道联邦服务的URL。[`stellar.toml`文件](../ concepts/stellar-toml.md)是公开可用的文件，其他人可以在其中找到有关恒星集成的信息。它应始终存储在：

`https://[YOUR DOMAIN]/.well-known/stellar.toml`

它可以列出很多属性，但我们现在关心的是联邦服务的URL。您的`stellar.toml`文件应该类似于：

<code-example name="stellar.toml">

```toml
FEDERATION_SERVER = "https://www.your_org.com:8002/federation"
```

</code-example>

联邦服务的实际URL您可以任意配置——它可以位于您的`www`子域及其它路径上，也可以使用不同的端口，甚至可以使用不同的域名。**但是，必须配置HTTPS，并使用有效的SSL证书。** [^ssl]


## 发送联邦查询

可以通过发送HTTP请求来测试联邦服务

<code-example name="Request a Federation Info">

```bash
curl "https://www.your_org.com:8002/federation?q=tunde_adebayo*your_org.com&type=name"
```

```js
var request = require('request');

request.get({
  url: 'https://www.your_org.com:8002/federation',
  qs: {
    q: 'tunde_adebayo*your_org.com',
    type: 'name'
  }
}, function(error, response, body) {
  console.log(body);
});
```

```java
import org.apache.http.HttpEntity;
import org.apache.http.HttpResponse;
import org.apache.http.client.HttpClient;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.util.EntityUtils;
import org.apache.http.client.utils.URIBuilder;
import java.net.URI;

class FederationRequest {
  public static void main(String [] args) throws Exception {
    URI federationUri = new URIBuilder("https://www.your_org.com:8002/federation")
      .addParameter("q", "tunde_adebayo*your_org.com")
      .addParameter("type", "name")
      .build();

    HttpGet federationRequest = new HttpGet(federationUri);
    HttpClient httpClient = HttpClients.createDefault();
    HttpResponse response = httpClient.execute(federationRequest);
    HttpEntity entity = response.getEntity();
    if (entity != null) {
      String body =  EntityUtils.toString(entity);
      System.out.println(body);
    }
  }
}
```

</code-example>

你应得到以下响应：

```json
{
  "stellar_address": "tunde_adebayo*your_org.com",
  "account_id": "GAIGZHHWK3REZQPLQX5DNUN4A32CSEONTU6CMDBO7GDWLPSXZDSYA4BU",
  "memo_type": "text",
  "memo": "tunde_adebayo"
}
```

<nav class="sequence-navigation">
  <a rel="prev" href="2-bridge-server.md">上一章节：桥接服务</a>
  <a rel="next" href="4-compliance-server.md">下一章节：合规服务</a>
</nav>


[^friendly_names]: 联邦地址使用`*`来区分用户名和域名，所以可以使用电邮地址作为用户名。如 `amy@gmail.com*your_org.com`。

[^federation_tables]: 如果你想让联邦服务可以覆盖多个域名，那么需要有一列来存储域名。

[^ssl]: 公共服务需要SSL证书来保证安全。测试期间，你可以从 http://letsencrypt.org 获取到免费证书。你也可以自行签署证书，但需要自行在测试环境中部署该证书。
