---
title: Stellar.toml
---

# 介绍

`stellar.toml` 是用于发布网站与 Stellar 网络集成信息的地方，任何网站都可以发布 Stellar 网络信息。在这里你可以声明你的校验公钥、[联邦服务器地址](./federation.md) 、正在运行对等节点(peers)、仲裁集合(quorum set)，如果你是一个锚点的话，你还需要添加一些其它信息。

stellar.toml 文件以 [TOML 格式](https://github.com/toml-lang/toml)编码。

## 发布 stellar.toml

对于给定的域名 "DOMAIN"，stellar.toml 文件应该能从以下位置访问到：

`https://DOMAIN/.well-known/stellar.toml`

## 开启跨域资源共享(CORS)
您必须为 stellar.toml 文件启用 CORS，这样人们才能从其他站点访问这个文件。 也就是说您*必须*将 stellar.toml 文件的 HTTP 响应头设置为如下：

```
Access-Control-Allow-Origin: *
```

**重要**: 若想只为 stellar.toml 文件启用 CORS(或其它任任何文件)。在 Apache 中你可以这样设置：

```xml
<Location "/.well-known/stellar.toml">
    Header set Access-Control-Allow-Origin "*"
</Location>
```

在 Nginx 中你可以这样设置：

```
location /.well-known/stellar.toml {
 add_header 'Access-Control-Allow-Origin' '*';
}
```

对于其它类型的服务器，请查阅：http://enable-cors.org/server.html

## 测试 CORS

1. 在终端中允许以下命令 (将 stellar.org 替换为你自己的域名)：

  ```bash
  curl --head https://stellar.org/.well-known/stellar.toml
  ```

2. 确定 `Access-Control-Allow-Origin` 出现在 HTTP 头中

  ```bash
  curl --head https://stellar.org/.well-known/stellar.toml
  HTTP/1.1 200 OK
  Accept-Ranges: bytes
  Access-Control-Allow-Origin: *
  Content-length: 482
  ...
  ```

3. 使用上述命令测试未启用 CORS 的页面，确认 `Access-Control-Allow-Origin` 头没有出现。

## Stellar.toml 样例

文件需要是 UTF-8 编码，换行符可以是 LF 或 CRLF。
空行和以 `#` 开头的行会被忽略。
未定义的字段会被保留。
所有的字段都是可选的。
以下配置中的许多字段和您的 [stellar-core.cfg](https://github.com/stellar/stellar-core/blob/master/docs/stellar-core_example.cfg) 配置有关。

```toml
# Sample stellar.toml

#   The endpoint which clients should query to resolve stellar addresses
#   for users on your domain.
FEDERATION_SERVER="https://api.stellar.org/federation"

# The endpoint used for the compliance protocol
AUTH_SERVER="https://api.stellar.org/auth"

# The signing key is used for the compliance protocol
SIGNING_KEY="GBBHQ7H4V6RRORKYLHTCAWP6MOHNORRFJSDPXDFYDGJB2LPZUFPXUEW3"

# convenience mapping of common names to node IDs.
# You can use these common names in sections below instead of the less friendly nodeID.
# This is provided mainly to be compatible with the stellar-core.cfg
NODE_NAMES=[
"GD5DJQDDBKGAYNEAXU562HYGOOSYAEOO6AS53PZXBOZGCP5M2OPGMZV3  lab1",
"GB6REF5GOGGSEHZ3L2YK6K4T4KX3YDMWHDCPMV7MZJDLHBDNZXEPRBGM  donovan",
"GBGR22MRCIVW2UZHFXMY5UIBJGPYABPQXQ5GGMNCSUM2KHE3N6CNH6G5  nelisky1",
"GDXWQCSKVYAJSUGR2HBYVFVR7NA7YWYSYK3XYKKFO553OQGOHAUP2PX2  jianing",
"GAOO3LWBC4XF6VWRP5ESJ6IBHAISVJMSBTALHOQM2EZG7Q477UWA6L7U  anchor"
]

#   A list of accounts that are controlled by this domain.
ACCOUNTS=[
"$sdf_watcher1",
"GAENZLGHJGJRCMX5VCHOLHQXU3EMCU5XWDNU4BGGJFNLI2EL354IVBK7"
]

#   Any validation public keys that are declared
#   to be used by this domain for validating ledgers and are
#   authorized signers for the domain.
OUR_VALIDATORS=[
"$sdf_watcher2",
"GCGB2S2KGYARPVIA37HYZXVRM2YZUEXA6S33ZU5BUDC6THSB62LZSTYH"
]

# DESIRED_BASE_FEE (integer)
# This is what you would prefer the base fee to be. It is in stroops.
DESIRED_BASE_FEE=100

# DESIRED_MAX_TX_PER_LEDGER (integer)
# This is how many maximum transactions per ledger you would like to process.
DESIRED_MAX_TX_PER_LEDGER=400

#   List of IPs of known stellar-core's.
#   These are IP:port strings.
#   Port is optional.
#   By convention, IPs are listed from most to least trusted, if that information is known.
KNOWN_PEERS=[
"192.168.0.1",
"core-testnet1.stellar.org",
"core-testnet2.stellar.org:11290",
"2001:0db8:0100:f101:0210:a4ff:fee3:9566"
]

# list of history archives maintained by this domain
HISTORY=[
"http://history.stellar.org/prd/core-live/core_live_001/",
"http://history.stellar.org/prd/core-live/core_live_002/",
"http://history.stellar.org/prd/core-live/core_live_003/"
]

#   This section allows an anchor to declare currencies it currently issues.
#   Can be used by wallets and clients to trust anchors by domain name
[[CURRENCIES]]
code="USD"
issuer="GCZJM35NKGVK47BB4SPBDV25477PZYIYPVVG453LPYFNXLS3FGHDXOCM"
display_decimals=2 # Specifies how many decimal places should be displayed by clients to end users.

[[CURRENCIES]]
code="BTC"
issuer="$anchor"
display_decimals=7 # Maximum decimal places that can be represented is 7

# asset with meta info
[[CURRENCIES]]
code="GOAT"
issuer="GD5T6IPRNCKFOHQWT264YPKOZAWUMMZOLZBJ6BNQMUGPWGRLBK3U7ZNP"
display_decimals=2
name="goat share"
desc="1 GOAT token entitles you to a share of revenue from Elkins Goat Farm."
conditions="There will only ever be 10,000 GOAT tokens in existence. We will distribute the revenue share annually on Jan. 15th"
image="https://pbs.twimg.com/profile_images/666921221410439168/iriHah4f.jpg"

#   Potential quorum set of this domain's validators.
[QUORUM_SET]
VALIDATORS=[
"$self", "$lab1", "$nelisky1","$jianing",
"$eno","$donovan"
]

# optional extra information for humans
# Useful place for anchors to detail various policies and required info

###################################
# Required compliance fields:
#      name=<recipient name>
#      addr=<recipient address>
# Federation Format:
#        <phone number>*anchor.com
#        Forwarding supported by sending to: forward*anchor.com
#           forward_type=bank_account
#           swift=<swift code of receiving bank>
#           acct=<recipient account number at receiving bank>
# Minimum Amount Forward: $2 USD
# Maximum Amount Forward: $10000 USD


```
