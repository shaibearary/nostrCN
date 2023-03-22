NIP-28
======

公共聊天
-----------

 `draft` `optional` `author:ChristopherDavid` `author:fiatjaf` `author:jb55` `author:Cameri`

该 NIP 为公共聊天频道、频道消息和基本客户端审核定义了新的事件类型。

它保留了五种事件类型（40-44）以供立即使用：

- `40-创建频道 `
- `41-通道元数据 `
- `42-通道消息 `
- `43-隐藏消息 `
- `44-静音用户 `

以客户端为中心的审核使客户端开发人员可以自行决定他们希望在应用程序中包含哪些类型的内容，同时不会对中继提出额外的要求。

## 第 40 类：创建频道

创建公共聊天频道。

在“频道创建 `content`”字段中，客户端应包括基本频道元数据（ `name`， `about`， `picture` 如 Kind 41 中所指定）。

```json
{
    "content": "{\"name\": \"Demo Channel\", \"about\": \"A test channel.\", \"picture\": \"https://placekitten.com/200/200\"}",
    ...
}
```


## 41 类：设置频道元数据

更新频道的公共元数据。

客户端和中继应处理与 Kind 0 `metadata` 事件类似的 Kind 41 事件。

客户端应该忽略来自公开密钥的类型 41，而不是类型 40 公开密钥。

客户端应支持基本元数据字段：

-  `name` -字符串-频道名称
-  `about` -字符串-通道描述
-  `picture` -string-频道图片的 URL

客户端可以添加其他元数据字段。

客户应使用[NIP-10](10.md)标有“E ”的标签来推荐继电器。

```json
{
    "content": "{\"name\": \"Updated Demo Channel\", \"about\": \"Updating a test channel.\", \"picture\": \"https://placekitten.com/201/201\"}",
    "tags": [["e", <channel_create_event_id>, <relay-url>]],
    ...
}
```


## 类型 42：创建通道消息

向频道发送文本消息。

客户端应该使用[NIP-10](10.md)标记为“e ”的标签来推荐中继，并指定它是回复消息还是根消息。

客户应[NIP-10](10.md)将“p ”标记附加到回复中。

根消息：

```json
{
    "content": <string>,
    "tags": [["e", <kind_40_event_id>, <relay-url>, "root"]],
    ...
}
```

回复另一封邮件：

```json
{
    "content": <string>,
    "tags": [
        ["e", <kind_40_event_id>, <relay-url>, "root"],
        ["e", <kind_42_event_id>, <relay-url>, "reply"],
        ["p", <pubkey>, <relay-url>],
        ...
    ],
    ...
}
```


## 第 43 类：隐藏消息

用户不再希望看到某个消息。

 `content` 可以可选地包括元数据，例如 `reason`。

如果存在来自给定用户的与事件 42 `id` 匹配的事件 43，则客户端应当隐藏向该用户示出的事件 42。

客户端可以为除了发送事件 43 的用户之外的其他用户隐藏事件 42。

（例如，如果三个用户“隐藏”了一个原因包含“色情”一词的事件，则作为 IOS 应用程序的 NOSTR 客户端可以选择对所有 IOS 客户端隐藏该消息。）

```json
{
    "content": "{\"reason\": \"Dick pic\"}",
    "tags": [["e", <kind_42_event_id>]],
    ...
}
```

## 类型 44：静音用户

用户不再希望看到来自其他用户的消息。

 `content` 可以可选地包括元数据，例如 `reason`。

如果存在来自给定用户的与事件 42 `pubkey` 匹配的事件 44，则客户端应当隐藏向该用户示出的事件 42。

客户端可以为除了发送事件 44 的用户之外的用户隐藏事件 42。

```json
{
    "content": "{\"reason\": \"Posting dick pics\"}",
    "tags": [["p", <pubkey>]],
    ...
}
```

## NIP-10 继电器建议

对于[NIP-10](10.md)中继建议，客户端通常应使用原始（最旧）类型 40 事件的中继 URL.

客户端可以推荐任何中继 URL.例如，如果托管通道的原始种类 40 事件的中继离线，则客户端可以改为从备份中继或客户端比原始中继更信任的中继获取通道数据。


动机
----------
如果我们要解决社交媒体的抗审查通信问题，我们也可以解决电报式消息传递的问题。

我们可以把全球对话从围墙花园带到一个真正的公共广场，向所有人开放。


其他信息
---------------

- [与 FIATJAF 聊天演示公关 +JB55 评论] （https：//github.com/arcadecity/arcade/pull/28）
- [关于 NIP16 的对话] （https：//t.me/nostr_protocol/29566）
