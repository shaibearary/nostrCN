NIP-23
======

长格式内容
-----------------

 `draft` `optional` `author:fiatjaf`

该 NIP 定义了 `kind:30023`（根据 NIP-33 的参数化可替换事件）长格式文本内容，通常称为“文章”或“博客帖子”。

不应期望主要 `kind:1` 处理票据的“社交”客户实施此 NIP.

### 格式

这些事件的 `.content` 应该是 Markdown 语法中的字符串文本。

### 元数据

根据 NIP-12，对于最后一次更新 `.created_at` 的日期，应使用字段，对于“标签”/“标签”（即事件可能相关的主题） `"t"`，应使用事件标签。

可以根据需要将其他元数据字段作为标记添加到事件中。在这里，我们对 4 个可能有用的标准进行了标准化，尽管它们仍然是严格可选的：

-  `"title"`，用于文章标题
-  `"image"`，用于指向要与标题一起显示的图像的 URL.
-  `"summary"`，以获取文章摘要
-  `"published_at"`，获取文章首次发布的时间戳（以 UNIX 秒（字符串化）为单位）。

### 可编辑性

这些文章是可编辑的，因此它们应该利用 NIP-33 的可替换性功能，并包含一个 `"d"` 带有文章标识符的标签。客户端应注意仅从实现该功能的中继发布和读取这些事件。如果他们不这样做，他们也应该注意隐藏他们可能收到的同一篇文章的旧版本。

### 链接

可以使用 NIP-19 `naddr` 代码和 `"a"` 标签（参见 NIP-33 和 NIP-19）链接到物品。

### 参考文献

支持发布 NIP-23 事件的客户端应实现对解析粘贴的 NIP-19 `naddr` 标识符并将其自动添加到事件列表的 `.tags` 支持，以与 NIP-08 相同的方式将实际内容替换为字符串 `#[tag_index]` --或者，如果引用是 URL 的形式（例如， `[click here](naddr1...)`），那么它们应该直接替换为标签号，就像该名称的链接存在于 Markdown 的底部一样（例如， `[click here][0]`）。

Reader 客户端应解析 Markdown 并将这些引用替换为内部链接，以便可以使用 NIP-21 `nostr:naddr1...` 链接或指向将处理这些引用的 Web 客户端的直接链接直接访问引用的事件。

这里的想法是，有了这些标签，当一篇文章提到另一篇文章时，读者客户端可以在底部显示反向引用列表。

同样的原则也适用于 `nevent1...`、 `note1...` `nprofile1...` 或 `npub1...`。

## 示例事件

```json
{
  "kind": 30023,
  "created_at": 1675642635,
  "content": "Lorem [ipsum][4] dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.\n\nRead more at #[3].",
  "tags": [
    ["d", "lorem-ipsum"],
    ["title", "Lorem Ipsum"],
    ["published_at", "1296962229"],
    ["t", "placeholder"],
    ["e", "b3e392b11f5d4f28321cedd09303a748acfd0487aea5a7450b3481c60b6e4f87", "wss://relay.example.com"],
    ["a", "30023:a695f6b60119d9521934a691347d9f78e8770b56da16bb255ee286ddf9fda919:ipsum", "wss://relay.nostr.org"]
  ],
  "pubkey": "...",
  "id": "..."
}
```
