---
title: XDR
---

**XDR**，也被称为*外部数据表示法(External Data Representation)*，它被运用在 Stellar 网络和协议中。总账、事务、事务结果、历史，甚至在运行节点的计算机之间传递的消息都是使用 XDR 进行编码的。

XDR 标准在 [RFC 4506](http://tools.ietf.org/html/rfc4506.html) 被确定，它类似于 Protocol Buffers 或 Thrift。XDR 包含了一些重要特性:

- 它非常紧凑，因此可以快速传输并节约磁盘空间。
- 用 XDR 编码的数据是可靠和可预测的。字段总是按相同的顺序排列，这使得对 XDR 编码的消息进行加密、签名和验证变得简单。
- XDR 定义包含了对数据类型和结构的丰富描述，这在 JSON、TOML 或 YAML 等更简单的格式中是不可能完成的。

由于 XDR 是一种二进制格式，并不像 JSON 之类的简单格式那样广为人知，所以 Stellar SDK 包含了解析 XDR 的工具，当你使用 SDK 处理数据时，它会自动帮你完成解析。

此外，Horizo​​n API 服务器通常以 JSON 格式展示 XDR 数据中最重要的那部分数据，因此当您不使用 SDK 的时候，JSON 数据更容易解析。XDR 数据仍然包含在 JSON 中（编码为 base64 字符串），你可以在需要的时候读取它。

## .X 文件

XDR 中的数据结构在*接口定义文件*(IDL)中指定。用于 Stellar 网络的 IDL 文件可以在 [GitHub](https://github.com/stellar/stellar-core/tree/master/src/xdr) 上获取。