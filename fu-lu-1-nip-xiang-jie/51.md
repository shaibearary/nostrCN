NIP-51
======

列表
-------------------------

 `draft` `optional` `author:fiatjaf` `author:arcbtc` `author:monlovesmango` `author:eskema` `depends:33`

“列表”事件被定义为具有公共和/或私有标签的列表。公共标签将在事件 `tags` 中列出。私人标签将在事件 `content` 中加密。私有标签的加密将使用[NIP-04-加密直接消息](04.md)加密，使用列表作者的私钥和公钥作为共享密钥。应为创建的每个列表类型使用不同的事件种类。

如果列表类型只应为每个用户定义一次（如“静音”列表），则列表类型的事件应遵循的规范[NIP-16-可替换事件](16.md)。这些列表可以称为“可替换列表”。

否则，列表类型的事件应遵循的规范[NIP-33-参数化可替换事件](33.md)，其中列表名称将用作“d ”参数。这些列表可以称为“参数化可替换列表”。

## 可替换列表事件示例

假设用户想要创建一个“静音”列表，并具有以下键：
```
priv: fb505c65d4df950f5d28c9e4d285ee12ffaf315deef1fc24e3c7cd1e7e35f2b1
pub: b1a5c93edcc8d586566fde53a20bdb50049a97b15483cb763854e57016e0fa3d
```
用户希望公开包括这些用户：

```json
["p", "3bf0c63fcb93463407af97a5e5ee64fa883d107ef9e558472c4eb9aaaefa459d"],
["p", "32e1827635450ebb3c5a7d12c1f8e7b2b514439ac10a67eef3d9fd9c5c68e245"]
```
并私下包括这些用户（下面是将被加密并放置在事件内容中的 JSON）：

```json
[
    ["p", "9ec7a778167afb1d30c4833de9322da0c08ba71a69e1911d5578d3144bb56437"],
    ["p", "8c0da4862130283ff9e67d889df264177a508974e2feb96de139804ea66d6168"]
]
```

然后，用户将创建一个“静音”列表事件，如下所示：

```json
{
  "kind": 10000,
  "tags": [
    ["p", "3bf0c63fcb93463407af97a5e5ee64fa883d107ef9e558472c4eb9aaaefa459d"],
    ["p", "32e1827635450ebb3c5a7d12c1f8e7b2b514439ac10a67eef3d9fd9c5c68e245"],
  ],
  "content": "VezuSvWak++ASjFMRqBPWS3mK5pZ0vRLL325iuIL4S+r8n9z+DuMau5vMElz1tGC/UqCDmbzE2kwplafaFo/FnIZMdEj4pdxgptyBV1ifZpH3TEF6OMjEtqbYRRqnxgIXsuOSXaerWgpi0pm+raHQPseoELQI/SZ1cvtFqEUCXdXpa5AYaSd+quEuthAEw7V1jP+5TDRCEC8jiLosBVhCtaPpLcrm8HydMYJ2XB6Ixs=?iv=/rtV49RFm0XyFEwG62Eo9A==",
  ...other fields
}
```


## 参数化可替换列表事件示例

假设用户想要创建人员的“分类人员”列表 `nostr`，并且具有以下键：
```
priv: fb505c65d4df950f5d28c9e4d285ee12ffaf315deef1fc24e3c7cd1e7e35f2b1
pub: b1a5c93edcc8d586566fde53a20bdb50049a97b15483cb763854e57016e0fa3d
```
用户希望公开包括这些用户：

```json
["p", "3bf0c63fcb93463407af97a5e5ee64fa883d107ef9e558472c4eb9aaaefa459d"],
["p", "32e1827635450ebb3c5a7d12c1f8e7b2b514439ac10a67eef3d9fd9c5c68e245"]
```
并私下包括这些用户（下面是将被加密并放置在事件内容中的 JSON）：

```json
[
    ["p", "9ec7a778167afb1d30c4833de9322da0c08ba71a69e1911d5578d3144bb56437"],
    ["p", "8c0da4862130283ff9e67d889df264177a508974e2feb96de139804ea66d6168"]
]
```

然后，用户将创建一个“分类人员”列表事件，如下所示：

```json
{
  "kind": 30000,
  "tags": [
    ["d", "nostr"],
    ["p", "3bf0c63fcb93463407af97a5e5ee64fa883d107ef9e558472c4eb9aaaefa459d"],
    ["p", "32e1827635450ebb3c5a7d12c1f8e7b2b514439ac10a67eef3d9fd9c5c68e245"],
  ],
  "content": "VezuSvWak++ASjFMRqBPWS3mK5pZ0vRLL325iuIL4S+r8n9z+DuMau5vMElz1tGC/UqCDmbzE2kwplafaFo/FnIZMdEj4pdxgptyBV1ifZpH3TEF6OMjEtqbYRRqnxgIXsuOSXaerWgpi0pm+raHQPseoELQI/SZ1cvtFqEUCXdXpa5AYaSd+quEuthAEw7V1jP+5TDRCEC8jiLosBVhCtaPpLcrm8HydMYJ2XB6Ixs=?iv=/rtV49RFm0XyFEwG62Eo9A==",
  ...other fields
}
```

## 列出事件种类

| kind   |列表类型|
| ------ | ----------------------- |
| 10000  |静音|
| 10001  |别针|
| 30000  |分类的人|
| 30001  |分类书签|

### 静音列表

具有种类 `10000` 的事件被定义为用于列出用户想要静音的内容的可替换列表事件。任何标准化标签都可以包括在静音列表中。

### 端号列表

带有类型 `10001` 的事件定义为可替换列表事件，用于列出用户想要固定的内容。任何标准化标记都可以包含在端号列表中。

### 分类的人员列表

具有种类 `30000` 的事件被定义为用于对人进行分类的参数化可替换列表事件。此事件的“d”参数保存列表的类别名称。这些列表中包含的标记必须遵循中[NIP-02-联系人列表和宠物名称](02.md)定义的第 3 类事件的格式。

### 已分类的书签列表

带有类型 `30001` 的事件被定义为参数化的可替换列表事件，用于对书签进行分类。此事件的“d”参数保存列表的类别名称。任何标准化的标签都可以包含在分类的书签列表中。
