咬：26
=======

委托事件签名
-----

 `draft` `optional` `author:markharding` `author:minds`

该 NIP 定义了如何委托事件，以便它们可以由其他密钥对签名。

该提议的另一个应用是在与客户端交互时抽象出“根”密钥对的使用。例如，用户可以为他们希望使用的每个客户端生成新的密钥对，并授权这些密钥对代表他们的根 pubkey 生成事件，其中根密钥对存储在冷存储中。

#### 引入“授权”标签

此 NIP 引入了一个新标签： `delegation`，其格式如下：

```json
[
  "delegation",
  <pubkey of the delegator>,
  <conditions query string>,
  <delegation token: 64-byte Schnorr signature of the sha256 hash of the delegation string>
]
```

##### 委派令牌

**委派令牌**应该是以下字符串的 SHA256 哈希的 64 字节 Schnorr 签名：

```
nostr:delegation:<pubkey of publisher (delegatee)>:<conditions query string>
```

##### 条件查询字符串

上述查询字符串中支持以下字段和运算符：

*菲尔兹*:
1. ` 种类 `
   -  *运营商*:
      -   `=${KIND_NUMBER}` -被委派者只能签署此类事件
2. `_ 创建于 `
   -  *运营商*:
      -   `<${TIMESTAMP}` -被委派者只能为在指定时间戳内创建***以前***的事件签名
      -   `>${TIMESTAMP}` -被委派者只能为在指定时间戳内创建***之后***的事件签名

要创建单个条件，必须使用受支持的字段和运算符。可以在单个查询字符串中使用多个条件，包括在同一字段上。条件必须结合 `&`。

例如，以下条件字符串是有效的：

- `kind=1&_ 创建于 <1675721813`
- `kind=0&kind=1&_ 创建于>1675721813`
- `KIND=1& 在>1674777689 创建了 _& 在 <1675721813 创建了 _`

对于绝大多数用例，建议查询字符串应包含 `created_at` ***之后***反映当前时间的条件，以防止被委派者代表委派者发布历史记录。

#### 例子

```
# Delegator:
privkey: ee35e8bb71131c02c1d7e73231daa48e9953d329a4b701f7133c8f46dd21139c
pubkey:  8e0d3d3eb2881ec137a11debe736a9086715a8c8beeeda615780064d68bc25dd

# Delegatee:
privkey: 777e4f60b4aa87937e13acc84f7abcc3c93cc035cb4c1e9f7a9086dd78fffce1
pubkey:  477318cfb5427b9cfc66a9fa376150c1ddbc62115ae27cef72417eb959691396
```

在给定当前时间戳为 `1674834236` 的情况下，从现在起的 30 天内向被委派者（477318CF）授予注释发布授权的委派字符串。
```json
nostr:delegation:477318cfb5427b9cfc66a9fa376150c1ddbc62115ae27cef72417eb959691396:kind=1&created_at>1674834236&created_at<1677426236
```

然后，委托者（8e0d3d3e）对上述委托字符串的 SHA256 散列进行签名，其结果是委托令牌：
```
6f44d7fe4f1c09f3954640fb58bd12bae8bb8ff4120853c4693106c82e920e2b898f1f9ba9bd65449a987c39c0423426ab7b53910c0c6abfb41b30bc16e5f524
```

被委派者（477318CF）现在可以代表委派者（8E0D3D3E）构造事件。然后，被委派者使用自己的私钥对事件进行签名并发布。
```json
{
  "id": "e93c6095c3db1c31d15ac771f8fc5fb672f6e52cd25505099f62cd055523224f",
  "pubkey": "477318cfb5427b9cfc66a9fa376150c1ddbc62115ae27cef72417eb959691396",
  "created_at": 1677426298,
  "kind": 1,
  "tags": [
    [
      "delegation",
      "8e0d3d3eb2881ec137a11debe736a9086715a8c8beeeda615780064d68bc25dd",
      "kind=1&created_at>1674834236&created_at<1677426236",
      "6f44d7fe4f1c09f3954640fb58bd12bae8bb8ff4120853c4693106c82e920e2b898f1f9ba9bd65449a987c39c0423426ab7b53910c0c6abfb41b30bc16e5f524"
    ]
  ],
  "content": "Hello, world!",
  "sig": "633db60e2e7082c13a47a6b19d663d45b2a2ebdeaf0b4c35ef83be2738030c54fc7fd56d139652937cdca875ee61b51904a1d0d0588a6acd6168d7be2909d693"
}
```

如果满足条件（ `kind=1` 在此示例中为、 `created_at>1674834236` 和 `created_at<1677426236`），并且在验证委派令牌时，发现与原始委派字符串中的条件相比未发生变化，则应将事件视为有效委派。

客户端应显示委托注释，就像它是由委托者（8E0D3D3E）直接发布的一样。


#### 中继和客户端查询支持

中继应响应请求，例如 `["REQ", "", {"authors": ["A"]}]` 通过查询 `pubkey` 和委托标记值 `[1]`。