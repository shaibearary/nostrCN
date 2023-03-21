---
description: 原文：https://github.com/nostr-protocol/nips/blob/master/04.md
---

# NIP04

## 加密的直接消息

`final` `optional` `author:arcbtc`

带有 kind `4` 的特殊事件，表示"加密的直接消息"。它应该具有以下属性：

\*\* `content` \*\*必须等于用户想要写入的任何内容的 base64 编码、AES-256-CBC 加密字符串，使用接收方的公钥和发送方的私钥组合生成的共享密码进行加密；这由 Base64 编码的初始化向量附加，就好像它是一个名为" IV "的查询字符串参数。格式如下： `"content": "<encrypted_text>?iv=<initialization_vector>"`。

\*\* `tags` \*\*必须包含标识消息接收者的条目（以便中继可以自然地将此事件转发给它们），格式 `["p", "<pubkey, as a hex string>"]` 为。

\*\* `tags` \*\*可能包含一个条目，该条目标识对话中的前一条消息或我们明确回复的消息（以便可能发生上下文相关的、更有组织的对话），格式 `["e", "<event_id>"]` 为。

**注意**：默认情况下，在[libsecp256k1](https://github.com/bitcoin-core/secp256k1)ECDH 实现中，秘密是共享点的 SHA256 哈希（X 和 Y 坐标）。在 NOSTR 中，仅将共享点的 X 坐标用作秘密，并且不对其进行散列。如果使用 libsecp256k1，则必须将复制 X 坐标的自定义函数作为中 `secp256k1_ecdh` 的 `hashfp` 参数传递。看吧[here](https://github.com/bitcoin-core/secp256k1/blob/master/src/modules/ecdh/main\_impl.h#L29)。

在 JavaScript 中生成此类事件的代码示例：

```js
import crypto from 'crypto'
import * as secp from '@noble/secp256k1'

let sharedPoint = secp.getSharedSecret(ourPrivateKey, '02' + theirPublicKey)
let sharedX = sharedPoint.slice(1, 33)

let iv = crypto.randomFillSync(new Uint8Array(16))
var cipher = crypto.createCipheriv(
  'aes-256-cbc',
  Buffer.from(sharedX),
  iv
)
let encryptedMessage = cipher.update(text, 'utf8', 'base64')
encryptedMessage += cipher.final('base64')
let ivBase64 = Buffer.from(iv.buffer).toString('base64')

let event = {
  pubkey: ourPubKey,
  created_at: Math.floor(Date.now() / 1000),
  kind: 4,
  tags: [['p', theirPublicKey]],
  content: encryptedMessage + '?iv=' + ivBase64
}
```

## 安全警告

该标准在对等体之间的加密通信中并不被认为是最先进的，并且它会泄漏事件中的元数据，因此它不能用于你真正需要保密的任何事情，并且只能用于限制谁可以获取你的 `kind:4` 事件的中继 `AUTH`。
