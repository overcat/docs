---
title: 下一步
sequence:
  previous: 4-compliance-server.md
---

恭喜！如果您已经抵达这里，您现在应已拥有可用的桥接服务，联邦服务，和合规服务，可以在恒星网络上收发安全可控的付款了。

## 测试

收发需要您的服务器与其他人进行交互，测试可能变得困难。 鉴于此，恒星基金会构建了一个方便的测试工具，您可以使用我们的测试锚点进行测试收发。 它可以模拟各种故障情景，因此您可以确保处理所有极端情况。 在这里查看测试工具[gostellar.org](http://gostellar.org).

## 转移到生产环境

当您准备将服务转移到生产环境并支持恒星公有网络上的事务时，您应该确保仔细检查以下几点：

- 更新桥接服务和合规服务配置文件中的Horizon URL和网络密语（network passphrase）。 桥接服务将需要公有网络（而不是测试网络）上的horizon服务器来发送事务。 桥接服务和合规服务在其配置文件中也有`network_passphrase`选项。公有网络使用另外的密语：

    ```toml
    network_passphrase = "Public Global Stellar Network ; September 2015"
    ````

- 确保您的桥接服务和合规服务的内部端口不对外公开。这两个服务都允许特权操作，如果有一些不该有权限的人访问到了它们，可能付出的代价会非常昂贵。

## 接下来是？

虽然您现在已经学会了处理锚点的核心操作，但是运营锚点可能需要了解或应该考虑的事情还有很多：

- [运行自有节点和horizon服务](https://stellar.org/developers/stellar-core/software/admin.html)。这样做可以减少对其它供应商的依赖，并使整个恒星网络更加强大。
- 阅读本指南中[security](../security.md)章节。
- 学习[在分布式交易所上买卖资产](../concepts/exchange.md)。
- 使用[做市机器人来为您的资产注入自动化流动性](https://github.com/lightyeario/kelp)。
- 探索[多重签名系统](../concepts/multi-sig.md)保证关键账户更加安全。
- 使用[信道](../channels.md)来同时提交更多事务。
- 与其它恒星开发者聊天：[恒星Slack社区](http://slack.stellar.org/)
- 为恒星软件[贡献](../contributing.md) 您的补丁以及改进代码。

<nav class="sequence-navigation">
  <a rel="prev" href="4-compliance-server.md">上一章节：合规服务</a>
</nav>
