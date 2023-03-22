NIP-46
======

NOSTR 连接
------------------------

 `draft` `optional` `author:tiero` `author:giowe` `author:vforvalerio87`

## 基本原理

私钥应尽可能少地暴露给系统（应用程序、操作系统、设备），因为每个系统都会增加攻击面。

输入私钥也很烦人，并且需要将它们暴露给更多的系统，例如可能被恶意应用程序监视的操作系统剪贴板。


## 条款

* **应用程序**：代表 NOSTR 帐户的任何平台*要求*上的 NOSTR 应用程序。
* **签名者**：持有 NOSTR 帐户的私钥并*可以签名*代表该帐户的 NOSTR 应用程序。


## `TL；博士


**应用程序**并且**签名者**使用选择的中继，使用种类 `24133` 向彼此发送短暂的加密消息。

APP 提示签名者执行获取公钥或签名事件等操作。

 `content` 字段必须是加密的 JSONRPC-ISH**请求**或**响应**。

## 签名者协议

### 留言

#### 请求

```json
{
  "id": <random_string>,
  "method": <one_of_the_methods>,
  "params": [<anything>, <else>]
}
```

#### 回应

```json
{
  "id": <request_id>,
  "result": <anything>,
  "error": <reason>
}
```

### 方法


#### 强制性的

以下是远程签名者应用程序必须实现的强制方法：

- **描述**
  - 参数 []
  - 结果 `["describe", "get_public_key", "sign_event", "connect", "disconnect", "delegate",...]`
- **获取 _ 公共 _ 密钥**
  - 参数 []
  - 结果 `pubkey`
- 为**_ 事件**签名
  - 参数 [ `event`]
  - 结果 `event_with_signature`

#### 可选


- **连接**
  - 参数 [ `pubkey`]
- **断开连接**
  - 参数 []
- **代表**
  - 参数 [ `delegatee`， `{ kind: number, since: number, until: number}`]
  - 结果 `{ from: string, to: string, cond: string, sig: string}`
- **去拿 _ 继电器**
  - 参数 []
  - 结果 `{ [url: string]: {read: boolean, write: boolean}}`
- **NIP04_ 加密**
  - 参数 [ `pubkey`， `plaintext`]
  - 结果 `nip4 ciphertext`
- **NIP04_ 解密**
  - 参数 [ `pubkey`， `nip4 ciphertext`]
  - 结果 [ `plaintext`]


注意： `pubkey` 和 `signature` 是十六进制编码的字符串。


### NOSTR 连接 URI

通过扫描 QR 码、点击深层链接或复制粘贴 URI 来**签名者**发现**应用程序**。

**应用程序**生成一个特殊的 URI，其前缀 `nostrconnect://` 和基本路径使用以下 querystring 参数**URL已编码**进行十六进制编码 `pubkey`

- 连接且**签名者**必须发送和侦听消息的所选**应用程序**中继的 `relay` URL.
- 的**应用程序** `metadata` 元数据 JSON
    - 的**应用程序** `name` 人类可读名称
    -  `url`（可选）请求连接的网站的 URL
    -  `description`（可选）的**应用程序**说明
    -  `icons`（可选）的**应用程序**图标的 URL 数组。

#### JavaScript

```js
const uri = `nostrconnect://<pubkey>?relay=${encodeURIComponent("wss://relay.damus.io")}&metadata=${encodeURIComponent(JSON.stringify({"name": "Example"}))}`
```

#### 例子
```sh
nostrconnect://b889ff5b1513b641e2a139f661a661364979c5beee91842f8f0ef42ab558e9d4?relay=wss%3A%2F%2Frelay.damus.io&metadata=%7B%22name%22%3A%22Example%22%7D
```



## 流动

 `content` 字段包含由指定[NIP04](https://github.com/nostr-protocol/nips/blob/master/04.md)的加密消息。 `kind` 被选中的是 `24133`。

### 连接

1. 用户点击网站上的**“连接”**按钮或用二维码扫描
2. 它将显示一个 URI 来打开一个已启用的**签名者**“nostr connect ”
3. 在 URI 中有一个 IE 的**应用程序**公钥。 `nostrconnect://<pubkey>&relay=<relay>&metadata=<metadata>`
4. 他**签名者**将发送一条消息来确认 `connect` 请求，同时发送的还有他的公钥

### 断开连接（从应用程序）

1. 用户单击**“断开连接”**上的**应用程序**按钮
2. **应用程序**将向**签名者**发送一条带有 `disconnect` 请求的消息
3. **签名者**将发送消息以确认 `disconnect` 请求

### 断开（与签名者）

1. 用户单击**“断开连接”**上的**签名者**按钮
2. **签名者**将向**应用程序**发送一条带有 `disconnect` 请求的消息


### 获取公钥

1. **应用程序**将向**签名者**发送一条带有 `get_public_key` 请求的消息
3. **签名者**将发回一条包含公钥的消息作为对 `get_public_key` 请求的响应

### 签名事件

1. **应用程序**将向**签名者**发送一条消息，其中包含一个 `sign_event` 请求以及要签名的**事件**
2. **签名者**将向用户显示一个弹出窗口，以检查事件并对其签名
3.  `signature` 作为对 `sign_event` 请求的响应，**签名者**将发回一条包含事件（包括 `id` 和 schnorr）的消息

### 代表

1. **应用程序**将向**签名者**发送一条包含元数据的消息，其中包含一个 `delegate` 请求以及**条件**要委托的**应用程序**的查询字符串和**公共钥匙**。
2. **签名者**将向用户显示一个弹出窗口，以委托**应用程序**代表其进行签名
3. **签名者**将发回带有签名[NIP-26委派令牌](https://github.com/nostr-protocol/nips/blob/master/26.md)的消息或拒绝该消息


