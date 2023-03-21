NIP-19
======

BECH32 编码实体
-----------------------

 `draft` `optional` `author:jb55` `author:fiatjaf` `author:Semisol`

该 NIP 对 BECH32 格式的字符串进行了标准化，这些字符串可用于在客户端中显示密钥、ID 和其他信息。这些格式并不意味着在核心协议中的任何地方使用，它们仅用于向用户显示、复制粘贴、共享、呈现 QR 码和输入数据。

建议以十六进制或二进制格式存储 ID 和密钥，因为这些格式更接近于核心协议必须实际使用的格式。

## 裸键和 ID

以防止私钥、公钥和事件 ID 之间的混淆和混合，它们都是 32 字节的字符串。具有不同前缀的 BECH32-（NOT-M）编码可用于这些实体中的每一个。

以下是可能的 BECH32 前缀：

  -  `npub`：公钥
  -  `nsec`：私钥
  -  `note`：钞票 ID

示例：十六进制公钥 `3bf0c63fcb93463407af97a5e5ee64fa883d107ef9e558472c4eb9aaaefa459d` 转换为 `npub180cvv07tjdrrgpa0j7j7tmnyl2yr6yr7l8j4s3evf6u64th6gkwsyjh6w6`。

密钥和 ID 的 BECH32 编码并不意味着在标准 NIP-01 事件格式或过滤器中使用，它们仅用于更人性化的显示和输入。客户端目前仍应接受十六进制和 NPUB 格式的密钥，并在内部进行转换。

## 具有额外元数据的可共享标识符

当共享简档或事件时，应用可以决定包括中继信息和其他元数据，使得其他应用可以更容易地定位和显示这些实体。

对于这些事件，内容是（类型-长度-值）的 `TLV` 二进制编码列表，其中 `T` 和 `L` 各为 1 字节（ `uint8` 即，0-255 范围内的数字），并且 `V` 是由 `L` 指示的大小的字节序列。

以下是可能的 BECH32 前缀 `TLV`：

  -  `nprofile`：NOSTR 配置文件
  -  `nevent`：NOSTR 事件
  -  `nrelay`：一个 NOSTR 继电器
  -  `naddr` NOSTR 参数化可替换事件坐标（NIP-33）

这些可能的标准化 `TLV` 类型如下所示：

-  `0`: `special`
  - 取决于 BECH32 前缀：
    - 因为 `nprofile` 它将是配置文件公钥的 32 个字节
    - 因为 `nevent` 它将是事件 ID 的 32 字节
    - 对于 `nrelay`，这是中继 URL
    - 对于 `naddr`，它是被引用事件的标识符（ `"d"` 标记
-  `1`: `relay`
  - 对于 `nprofile`、 `nevent` 和 `naddr`、_可选地_，更有可能找到实体（配置文件或事件）的中继，编码为 ASCII
  - 这可以被包括多次。
-  `2`: `author`
  - 对于 `naddr`，事件的公钥的 32 个字节
  - 对于 `nevent`，_可选地_，事件的公钥的 32 个字节
-  `3`: `kind`
  - 对于 `naddr`，类型为 big-endian 的 32 位无符号整数

## 例子

-  `npub10elfcs4fr0l0r8af98jlmgdh9c8tcxjvz9qkw038js35mp4dma8qzvjptg` 应解码为十六进制 `7e7e9c42a91bfef19fa929e5fda1b72e0ebc1a4c1141673e2794234d86addf4e` 公钥，反之亦然
-  `nsec1vl029mgpspedva04g90vltkh6fvh240zqtv9k0t9af8935ke9laqsnlfe5` 应解码为私钥十六进制 `67dea2ed018072d675f5415ecfaed7d2597555e202d85b3d65ea4e58d2d92ffa`，反之亦然
-  `nprofile1qqsrhuxx8l9ex335q7he0f09aej04zpazpl0ne2cgukyawd24mayt8gpp4mhxue69uhhytnc9e3k7mgpz4mhxue69uhkg6nzv9ejuumpv34kytnrdaksjlyr9p` 应解码为具有以下 TLV 项目的配置文件：
  - 公钥： `3bf0c63fcb93463407af97a5e5ee64fa883d107ef9e558472c4eb9aaaefa459d`
  - 继电器： `wss://r.x.com`
  - 继电器： `wss://djbas.sadkb.com`

## 笔记

- 不得在 NIP-01 事件或 NIP-05 JSON 响应中使用 `npub` 密钥，仅支持十六进制格式。
