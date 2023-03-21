---
description: 原文：https://github.com/nostr-protocol/nips/blob/master/11.md
---

# NIP11

## 中继信息文档

`draft` `optional` `author:scsibug` `author:doc-hex` `author:cameri`

中继可以向客户端提供服务器元数据，以向它们通知能力、管理联系人和各种服务器属性。这是通过 HTTP 以 JSON 文档的形式提供的，与中继的 WebSocket 在相同的 URI 上。

当中继接收到对支持 WebSocket 升级的 URI 具有 `Accept` 标头 `application/nostr+json` 的 HTTP（S）请求时，它们应该返回具有以下结构的文档。

```json
{
  "name": <string identifying relay>,
  "description": <string with detailed information>,
  "pubkey": <administrative contact pubkey>,
  "contact": <administrative alternate contact>,
  "supported_nips": <a list of NIP numbers supported by the relay>,
  "software": <string identifying relay software URL>,
  "version": <string version identifier>
}
```

任何字段都可以省略，并且客户端必须忽略它们不理解的任何其他字段。中继必须通过发送 `Access-Control-Allow-Origin`、 `Access-Control-Allow-Headers` 和 `Access-Control-Allow-Methods` 报头来接受 CORS 请求。

## 字段描述

### 姓名

中继可以选择 `name` 在客户端软件中使用。这是一个字符串，应少于 30 个字符以避免客户端截断。

### 描述

有关中继的详细纯文本信息可能包含在 `description` 字符串中。建议不要包含用于自动换行的标记、格式或换行符，只需使用两个换行符来分隔段落。对长度没有限制。

### PubKey##\#

管理联系人可能以 `pubkey` 与 NOSTR 事件相同的格式列出（公钥为 `secp256k1` 32 字节十六进制）。如果列出了联系人，这将为客户端提供一个建议的地址，以便向系统管理员发送加密的直接消息（请参阅 `NIP-04`）。此地址的预期用途是报告滥用或非法内容、提交错误报告或请求其他技术帮助。

中继运营商没有义务对直接消息作出回应。

### 联系人

也可以在 `contact` 字段下列出备用联系人，其目的与 `pubkey` 相同。最好使用 NOSTR 公钥和直接消息。此字段的内容应该是一个 URI，使用或 `https` 等 `mailto` 方案为用户提供联系方式。

### 支持的 NIP##\#

随着 NOSTR 协议的发展，某些功能可能只能由实现特定 `NIP`。该字段是在中继中实现的 s 的 `NIP` 整数标识符的数组。示例包括 `1`、FOR `"NIP-01"` 和 `9` FOR `"NIP-09"`。客户端 `NIPs` 不应该被公布，并且可以被客户端忽略。

### 软件

可以在 `software` 属性中提供中继服务器实现。如果存在，这必须是项目主页的 URL.

### 版本

中继可以选择将其软件版本发布为字符串属性。字符串格式由中继实现定义。建议使用版本号或提交标识符。

## 额外字段

### 服务器限制

这些是中继对客户端施加的限制。你的客户应该预料到，超过这些_实际的_限制的请求会被拒绝或立即失败。

```json
{
...
  limitation: {
        max_message_length: 16384,
        max_subscriptions: 20,
        max_filters: 100,
        max_limit: 5000,
        max_subid_length: 100,
        min_prefix: 4,
        max_event_tags: 100,
        max_content_length: 8196,
        min_pow_difficulty: 30,
        auth_required: true,
        payment_required: true,
  }
...
}
```

* `max_message_length`：这是中继的传入 JSON 的最大字节数 将尝试解码并采取行动。当你发送大量订阅时，你将受到此值的限制。它还有效地限制了任何事件的最大大小。值从 `[` 到 `]` 计算，并且在 UTF-8 序列化之后（因此某些 Unicode 字符将占用 2-3 个字节）。它等于 WebSocket 消息帧的最大大小。
* `max_subscriptions`：可能的订阅总数 在到此中继的单个 WebSocket 连接上处于活动状态。与中继有（付费）关系的经过身份验证的客户端可能具有更高的限制。
* `max_filters`：每个订阅中筛选值的最大数目。 必须为 1 或更高。
* `max_subid_length`：订阅 ID 作为字符串的最大长度。
* `min_prefix`：对于 `authors` 要匹配的和 `ids` 过滤器 对于十六进制前缀，你必须在前缀中至少提供此数量的十六进制数字。
* `max_limit`：中继服务器会将每个过滤器的 `limit` 值限制为此数字。 这意味着客户端将无法从单个订阅筛选器中获取超过此数量的事件。这种箝位通常由继电器静默完成，但有了这个数字，你可以知道，如果你缩小滤波器的时间范围或其他参数，会有额外的结果。
* `max_event_tags`：在任何情况下，这都是列表中 `tags` 元素的最大数量。
* `max_content_length`：中 `content` 的最大字符数 任何事件的字段。这是 Unicode 字符的计数。在序列化为 JSON 之后，它可能会更大（以字节为单位），并且如果定义了，则仍然受 `max_message_length`。
* `min_pow_difficulty` 新事件将至少需要这种功率的困难， 基于[NIP-13](13.md)，否则它们将被此服务器拒绝。
* `auth_required`：此中继需要[NIP-42](42.md)身份验证 在新连接执行任何其他操作之前发生。即使设置为 False，特定操作也可能需要身份验证。
* `payment_required`：在新连接可以执行任何操作之前，此中继需要付费。

### 事件保留

可能存在与永久存储数据相关联的成本，因此中继可能希望声明保留时间。此处所述的值是未经身份验证的用户和访问者的默认值。付费用户可能会有其他政策。

保留时间以秒为单位， `null` 表示无限长。如果提供零，则这意味着根本不存储事件，并且优选地，当接收到这些事件时，将提供错误。

```json
{
...
  retention: [
    { kinds: [0, 1, [5, 7], [40, 49]], time: 3600 },
    { kinds: [[40000, 49999], time: 100 },
    { kinds: [[30000, 39999], count: 1000 },
    { time: 3600, count: 10000 }
  ]
...
}
```

`retention` 是一个规范列表：每个规范将适用于所有类型或类型的子集。可以将种类字段的范围指定为包含起始值和结束值的元组。然后，将指定类型（或所有）的事件限制为 AND `count` 或时间段。

通过将这些 `kind` 值的保留时间设为零，可以有效地将依赖于特定 `kind` 数字的基于 NOSTR 的协议列入黑名单。虽然这很不幸，但它确实允许客户端通过单个 HTTP 获取快速发现支持其协议的服务器。

无需为指定保留时间（如中[NIP-16](16.md)所定义），\_短暂的事件\_因为它们不会被保留。

### 内容限制

一些继电器可能受一个民族国家的任意法律管辖。这可能会限制哪些内容可以以明文形式存储在这些中继上。鼓励所有客户端使用加密来绕过此限制。

不可能描述每个国家的法律和政策的局限性，这些法律和政策本身通常是模糊和不断变化的。

因此，该字段允许中继运营商指示哪个国家的法律可能最终对其强制执行，然后间接地对其用户的内容强制执行。

用户应该能够避免在他们不喜欢的国家进行中继，和/或在更有利的地区选择中继。展现这种灵活性取决于客户端软件。

```json
{
...
  relay_countries: [ 'CA', 'US' ],
...
}
```

* `relay_countries`：两级 ISO 国家代码（ISO 3166-1 alpha-2）的列表，其法律和政策可能会影响此中继。 `EU` 可用于欧盟国家。

请记住，接力可能在一个国家举办，而不是拥有接力的法律实体所在的国家，因此很可能涉及多个国家。

### 社区首选项

至少对于公共文本笔记，接力可能会试图促进当地社区的发展。这将鼓励用户除了他们通常的个人关注之外，还关注该中继上的全球反馈。为了支持这一目标，继电器可以指定以下一些值。

```json
{
...
  language_tags: [ 'en', 'en-419' ],
  tags: [ 'sfw-only', 'bitcoin-only', 'anime' ],
  posting_policy: 'https://example.com/posting-policy.html',
...
}
```

* `language_tags` 是指示在中继上所说的主要语言的[IETF语言标签](https://en.wikipedia.org/wiki/IETF\_language\_tag)有序列表。
* `tags` 是要讨论的主题的限制列表。例如 `sfw-only`，表示在此中继上仅鼓励"安全工作"内容。这依赖于对"工作"和"社区"在谈论什么时感到"安全"的假设。随着时间的推移，一组通用的标签可能会出现，允许用户找到适合他们需要的中继，并且客户端软件将能够轻松地解析这些标签。 `bitcoin-only` 标签表示任何_替代币_，_“加密”或区块链_评论都将被毫不留情地嘲笑。
* `posting_policy` 是指向人类可读页面的链接，该页面指定了中继的社区策略。在这种 `sfw-only` 情况下，链接到一个页面是很重要的，这个页面涉及到你的发布政策的细节。

该 `description` 字段应用于简要描述你的社区目标和价值观。其他详细信息和法律条款 `posting_policy` 请参见。使用 `tags` 字段表示对内容或要讨论的主题的限制，这些内容或主题可以由相应的客户端软件进行机器处理。

### 付费中继

需要付款的中继可能希望公开其费用表。

\`\` \`JSON{ ... 支付 \_URL：" https：//my-relay/payments "，费用：{ }, ... }
