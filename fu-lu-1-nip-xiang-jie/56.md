NIP-56
======

报告
---------

 `draft` `optional` `author:jb55`

报告是 `kind 1984` 用于报告垃圾邮件、非法和明确内容的其他注释的注释。

该内容可以包含由报告该内容的实体提交的附加信息。

标签
----

报告事件必须包含 `p` 引用你要报告的用户的 pubkey 的标记。

如果报告注释， `e` 则还必须包括引用注释 ID 的标记。

必须包括一个 `report type` 字符串，作为所报告的 `e` 或 `p` 标记的第 3 个条目，该标记由以下报告类型组成：

-  `nudity` -对裸体、色情等的描述
-  `profanity` -亵渎、仇恨言论等
-  `illegal` -在某些司法管辖区可能是非法的
-  `spam` -垃圾邮件
-  `impersonation` -有人假扮别人

某些报告标记仅对配置文件报告有意义，例如 `impersonation`

示例事件
--------------

```json
{
  "kind": 1984,
  "tags": [
    [ "p", <pubkey>, "nudity"]
  ],
  "content": "",
  ...
}

{
  "kind": 1984,
  "tags": [
    [ "e", <eventId>, "illegal"],
    [ "p", <pubkey>]
  ],
  "content": "He's insulting the king!",
  ...
}

{
  "kind": 1984,
  "tags": [
    [ "p", <impersonator pubkey>, "impersonation"],
    [ "p", <victim pubkey>]
  ],
  "content": "Profile is imitating #[1]",
  ...
}
```

客户行为
---------------

如果客户愿意，他们可以使用朋友的报告来做出审核决定。例如，如果你的朋友中有 3 个以上的人报告个人资料是明确的，客户可以选择自动模糊所述帐户中的照片。


中继行为
--------------

不建议中继使用报告执行自动审核，因为它们很容易被利用。管理员可以使用来自受信任的版主的报告来删除非法或明确的内容，如果中继不允许这样做的话。
