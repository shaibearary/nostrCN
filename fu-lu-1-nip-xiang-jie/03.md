---
description: 原文：https://github.com/nostr-protocol/nips/blob/master/03.md
---

# NIP03

## OpenTimeStamps 事件证明

`draft` `optional` `author:fiatjaf`

当有可用的 OTS 时，它可能包含在现有事件主体中，位于以下 `ots` 项下：

```
{
  "id": ...,
  "kind": ...,
  ...,
  ...,
  "ots": <base64-encoded OTS file data>
}
```

\_事件ID\_必须用作要包含在 OpenTimeStamps Merkle 树中的原始哈希。

证明可以由中继自动提供（OTS 二进制内容仅附加到其接收的事件），也可以由客户端在首次将事件上传到中继时自行提供，并由客户端用于显示事件实际上"至少与 \[OTS 日期] 一样旧"。
