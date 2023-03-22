NIP-39
======

配置文件中的外部身份
-------------------------------

 `draft` `optional` `author:pseudozach` `author:Semisol`

## 抽象的

NOSTR 协议用户可能具有其他在线身份，例如用户名、配置文件页面、密钥对等。他们可能希望将这些数据包含在他们的配置文件元数据中，以便客户端可以解析、验证和显示这些信息。

## 元数据事件上的 `i` 标记

除了名称、关于、图片字段外，还为 `kind 0` 元数据事件内容引入了新的可选 `i` 标记，如[NIP-01](https://github.com/nostr-protocol/nips/blob/master/01.md)：
```json
{
    "id": <id>,
    "pubkey": <pubkey>,
    ...
    "tags": [
        ["i", "github:semisol", "9721ce4ee4fceb91c9711ca2a6c9a5ab"],
        ["i", "twitter:semisol_public", "1619358434134196225"],
        ["i", "mastodon:bitcoinhackers.org/@semisol", "109775066355589974"]
        ["i", "telegram:1087295469", "nostrdirectory/770"]
    ]
}
```

 `i` 标记有两个参数，定义如下：
1.  `platform:identity`：这是与 `:` 连接在一起的平台名称（例如 `github`）和该平台上的标识（例如 `semisol`）。
2.  `proof`：指向拥有此标识的证明的字符串或对象。

客户端应处理具有 2 个以上值的任何 `i` 标记，以便将来扩展。身份提供程序名称只能包括 `a-z`， `0-9` 并且不能包括 `:` 和字符 `._-/`。如果可能的话，应该通过用小写字母替换大写字母来规范化身份名称，并且如果一个实体有多个别名，则应该使用主要别名。

## 索赔类型

### `github`

身份：GitHub 用户名。

证据：一个 GitHub Gist ID.应使用包含文本 `Verifying that I control the following Nostr public key: <npub encoded public key>` 的单个文件创建 `<identity>` 此要点。这可以位于 `https://gist.github.com/<identity>/<proof>`。

### ` 推特 `

身份：Twitter 用户名。

证据：一个推特 ID.推文应由 `<identity>` 发布并包含文本 `Verifying my account on nostr My Public Key: "<npub encoded public key>"`。这可以位于 `https://twitter.com/<identity>/status/<proof>`。

### '乳齿象'

身份：格式 `<instance>/@<username>` 为的 Mastodon 实例和用户名。

证据：一个乳齿象的帖子 ID.这篇文章应该是由 `<username>@<instance>` 和有文字 `Verifying that I control the following Nostr public key: "<npub encoded public key>"` 发表。这可以位于 `https://<identity>/<proof>`。

### ` 电报 `

身份：电报用户 ID.

证明：格式 `<ref>/<id>` 为的字符串，指向在公共频道或组中发布的具有名称 `<ref>` 和消息 ID `<id>` 的消息。此消息应由用户 ID `<identity>` 发送，并具有文本 `Verifying that I control the following Nostr public key: "<npub encoded public key>"`。这可以位于 `https://t.me/<proof>`。
