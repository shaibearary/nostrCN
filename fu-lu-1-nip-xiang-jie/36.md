NIP-36
======

敏感内容/内容警告
-----------------------------------

 `draft` `optional` `author:fernandolguevara`

 `content-warning` 标签使用户能够指定事件的内容是否需要读者批准才能显示。客户端可以隐藏内容，直到用户对其进行操作。

#### 规格

```
tag: content-warning
options:
 - [reason]: optional  
```

#### 例子

```json
{
    "pubkey": "<pub-key>",
    "created_at": 1000000000,
    "kind": 1,
    "tags": [
      ["t", "hastag"],
      ["content-warning", "reason"] /* reason is optional */
    ],
    "content": "sensitive content with #hastag\n",
    "id": "<event-id>"
}
```
