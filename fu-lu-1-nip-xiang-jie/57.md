NIP-57
======

闪电击中
--------------

 `draft` `optional` `author:jb55` `author:kieran`

这个 NIP 定义了一种新的音符类型，称为闪电类型 `9735`。这些表示由名为的 `zapper` Lightning 节点发送的已支付的 Lightning 发票收据。我们还定义了另一种票据类型 `9734`，即 `zap request` 票据，这将在本文档中进行描述。

在 NOSTR 上拥有闪电收据允许客户显示来自网络上实体的闪电付款。这些可以用于娱乐或垃圾邮件威慑。


## 定义

 `zapper` -发送 ZAP 注释的 Lightning 节点或服务（种类 `9735`）

 `zap request` -一种 `9734` 由电击的人创建的注释

 `zap invoice` -从包含 `zap request` 注释的自定义 LNURL 端点获取的 BOLT11 发票


## 协议流

### 客户端

1. 从用户配置文件上的 LUD06 或 LUD16 字段计算用户的 LNURL 支付请求 URL

2. 获取 LNURL 支付请求静态端点（ `https://host.com/.well-known/lnurlp/user`）并收集 `allowsNostr` 和 `nostrPubkey` 字段。如果 `allowsNostr` 存在并且是 `true` `nostrPubkey` 有效的 BIP 340 公钥，则将该信息与用户相关联。 `nostrPubkey` 是的 `zapper` 公钥，用于授权发送给该用户的 ZAP.

3. 如果用户的 LNURL 支付请求端点支持 NOSTR，则客户端可以选择在每个帖子或用户配置文件上显示 Lightning Zap 按钮，客户端应生成一个 `zap invoice` 而不是普通的 LNURL 发票。

4. 要生成 `zap invoice`，请调用设置为 Milli-Satoshi 金额值的 `callback` URL `amount`。还必须 `nostr` 设置 QueryString 值。它是由用户密钥签名的 URI 编码 `zap request` 注释。 `zap request` 便笺包含 `e` 它正在删除的便笺的标记和 `p` 目标用户的公钥的标记。 `e` 标签是可选的，允许轮廓倾斜。可选 `a` 标签允许 Tipping 参数化可替换事件，如 NIP-23 长格式注释。 `zap request` 注释还必须有一个 `relays` 标签，该标签是从用户配置的中继中收集的。 `zap request` 票据应包含一个 `amount` 标签，该标签是 ZAP 的 Milli-Satoshi 值，客户应验证该值是否等于发票金额。这 `content` 可以是来自用户的附加评论，当在帖子和配置文件上列出 ZAP 时，可以显示该附加评论。

5. 支付此发票或将其传递给可以支付发票的应用程序。一旦它被支付，A `zap note` 将被创建 `zapper`。

### LNURL 服务器端

LNURL 服务器将需要一些额外的信息，以便客户端可以知道支持 ZAP 发票：

1. 将 a 添加 `nostrPubkey` 到 lnurl-pay 静态端点 `/.well-known/lnurlp/user`，其中 `nostrPubkey` 是创建 Zap Notes 的实体的 nostr pubkey `zapper`。客户端将使用它来授权 ZAP.

2. 添加一个 `allowsNostr` 字段并将其设置为 true.

3. 在 lnurl-pay 回调 URL 中，注意 `nostr` 查询字符串，其中注释的内容是 URI 编码 `zap request` 的 JSON.

4. 如果存在，必须验证 ZAP 请求注释：

	a. 它必须具有有效的 NOSTR 签名

	b. 它必须有标签。

	c. 它必须至少有一个 P 标签

	d. 它必须有 0 或 1 个电子标签

	e. 应该有一个 `relays` 标签与继电器发送 `zap` 通知。

	f. 如果有 `amount` 标记，它必须等于 `amount` 查询参数。

	g. 如果有 `a` 标记，则必须是有效的 NIP-33 事件坐标

5. 如果有效，则获取描述为此票据且仅此票据的描述哈希发票。描述中不包含其他 LNURL 元数据。

此时，一旦收到付款，Lightning 节点就准备好发送 ZAP 通知。

## ZAP 笔记

Zap Notes 是由闪电节点对已付发票作出反应而创建的。只有当发票描述（提交到描述哈希）包含 `zap request` 注释时，才会创建 ZAP 注释。

ZAP 注释示例：

```json
{
    "id": "67b48a14fb66c60c8f9070bdeb37afdfcc3d08ad01989460448e4081eddda446",
    "pubkey": "9630f464cca6a5147aa8a35f0bcdd3ce485324e732fd39e09233b1d848238f31",
    "created_at": 1674164545,
    "kind": 9735,
    "tags": [
      [
        "p",
        "32e1827635450ebb3c5a7d12c1f8e7b2b514439ac10a67eef3d9fd9c5c68e245"
      ],
      [
        "e",
        "3624762a1274dd9636e0c552b53086d70bc88c165bc4dc0f9e836a1eaf86c3b8"
      ],
      [
        "bolt11",
        "lnbc10u1p3unwfusp5t9r3yymhpfqculx78u027lxspgxcr2n2987mx2j55nnfs95nxnzqpp5jmrh92pfld78spqs78v9euf2385t83uvpwk9ldrlvf6ch7tpascqhp5zvkrmemgth3tufcvflmzjzfvjt023nazlhljz2n9hattj4f8jq8qxqyjw5qcqpjrzjqtc4fc44feggv7065fqe5m4ytjarg3repr5j9el35xhmtfexc42yczarjuqqfzqqqqqqqqlgqqqqqqgq9q9qxpqysgq079nkq507a5tw7xgttmj4u990j7wfggtrasah5gd4ywfr2pjcn29383tphp4t48gquelz9z78p4cq7ml3nrrphw5w6eckhjwmhezhnqpy6gyf0"
      ],
      [
        "description",
        "{\"pubkey\":\"32e1827635450ebb3c5a7d12c1f8e7b2b514439ac10a67eef3d9fd9c5c68e245\",\"content\":\"\",\"id\":\"d9cc14d50fcb8c27539aacf776882942c1a11ea4472f8cdec1dea82fab66279d\",\"created_at\":1674164539,\"sig\":\"77127f636577e9029276be060332ea565deaf89ff215a494ccff16ae3f757065e2bc59b2e8c113dd407917a010b3abd36c8d7ad84c0e3ab7dab3a0b0caa9835d\",\"kind\":9734,\"tags\":[[\"e\",\"3624762a1274dd9636e0c552b53086d70bc88c165bc4dc0f9e836a1eaf86c3b8\"],[\"p\",\"32e1827635450ebb3c5a7d12c1f8e7b2b514439ac10a67eef3d9fd9c5c68e245\"],[\"relays\",\"wss://relay.damus.io\",\"wss://nostr-relay.wlvs.space\",\"wss://nostr.fmt.wiz.biz\",\"wss://relay.nostr.bg\",\"wss://nostr.oxtr.dev\",\"wss://nostr.v0l.io\",\"wss://brb.io\",\"wss://nostr.bitcoiner.social\",\"ws://monad.jb55.com:8080\",\"wss://relay.snort.social\"]]}"
      ],
      [
        "preimage",
        "5d006d2cf1e73c7148e7519a4c68adc81642ce0e25a432b2434c99f97344c15f"
      ]
    ],
    "content": "",
    "sig": "b0a3c5c984ceb777ac455b2f659505df51585d5fd97a0ec1fdb5f3347d392080d4b420240434a3afd909207195dac1e2f7e3df26ba862a45afd8bfe101c2b1cc"
  }
```

* ZAP 便笺必须具有 `bolt11` 包含说明哈希螺栓 11 发票的标记。

* ZAP 票据必须包含 `description` 作为发票说明的标签。

*  `SHA256(description)` 必须与 BOLT11 发票中的描述哈希匹配。

* ZAP 票据可以包含 `preimage` 与 Bolt11 发票的支付散列匹配的。这不是真正的付款证明，没有真正的方法来证明发票是真实的或已经支付。你信任 ZAP 票据的作者，以确保付款的合法性。

ZAP 票据并不是付款凭证，它所能证明的只是某个 NOSTR 用户拿到了一张发票。ZAP 票据的存在意味着发票已支付，但考虑到流氓实现，它可能是一个谎言。


### 创建 Zap 便笺

当收到付款时，执行以下步骤：

1. 获取发票的说明。这需要在生成描述哈希发票的过程中保存在某个位置。它通过 CLN 自动保存，CLN 是这里使用的参考实现。

2. 将 Bolt11 描述解析为 JSON NOSTR 注释。你应该检查已解析注释的签名以确保其有效。这是 `zap request` 由正在切换的实体创建的注释。

4. 注释必须只有一个 `p` 标记

5. 注释必须有 0 或 1 个 `e` 标记

6. 创建包含 `p` 标记和可选 `e` 标记的 NOSTR 类型 `9735` 注释。内容应该是空的。创建 _ 的日期应设置为发票支付 _ 的日期，以确保等效性。

7. 将票据发送给发票描述中的 `zap request` 票据中声明的 `relays`。

Zapper 的参考实现如下：[Zapper][Zapper]

[zapper]: https://github.com/jb55/cln-nostr-zapper


## 客户行为

客户可以在帖子和个人资料上获取 ZAP 注释：

`{“种类”：[9735]，“#E ”：[..]}`

要授权这些注释，客户端必须从用户配置的 Lightning 地址或 LNURL 获取 `nostrPubkey`，并确保其帖子的 ZAP 是由此公钥创建的。如果客户不这样做，任何人都可以伪造未经授权的 ZAP.

一旦获得授权，客户可以在帖子上记录 ZAP，并将其列在个人资料中。如果 ZAP 请求注释包含非空 `content`，则可能显示 ZAP 注释。通常，客户端应向用户 `zap request` 显示注释，并使用 `zap note` 显示“ZAP Authorized by..”，但这是可选的。

## 未来的工作

通过对目标用户的 ZAP 请求注释进行加密，可以将 ZAP 扩展为更加私密，但为了简单起见，它已被排除在此初始草案之外。
