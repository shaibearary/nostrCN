NIP-13
======

工作证明
-------------

 `draft` `optional` `author:jb55` `author:cameri`

该 NIP 定义了一种为 NOSTR Notes 生成和解释工作证明的方法。工作量证明（Proof of Work，POW）是一种将计算工作量的证明添加到笔记的方法。这是一个承载证明，所有中继和客户端都可以使用少量代码进行普遍验证。这种证明可以作为垃圾邮件威慑的一种手段。

 `difficulty` 定义为 ID 中 `NIP-01` 的前导零位的数量。例如，的 `000000000e9d97a1ab09fc381030b346cdd7a142ad57e6df0b46dc9bef6c7e2d` ID 在 `36` 前导 0 位上有困难 `36`。

采矿
------

要为 `NIP-01` 便笺生成功率， `nonce` 需要使用标签：

```json
{"content": "It's just me mining my own business", "tags": [["nonce", "1", "20"]]}
```

挖掘时，更新 nonce 标记的第二个条目，然后重新计算 ID（请参阅[NIP-01](./01.md)）。如果 ID 具有所需数量的前导零位，则该音符已被挖掘。建议在此过程中也更新 `created_at`。

nonce 标签 `SHOULD` 的第三个条目包含目标难度。这使得客户端可以防止以较低难度为目标的批量垃圾邮件发送者幸运地匹配较高难度的情况。例如，如果你需要 40 位来回复你的线程，并且看到提交的目标为 30，则即使注释有 40 位的困难，你也可以安全地拒绝它。没有一个承诺的目标难度，你不能拒绝它。对目标难度的承诺是所有诚实的矿工都应该接受的，如果缺少难度承诺，客户 `MAY` 会拒绝与目标难度匹配的票据。

挖掘的注释示例
------------------

```json
{
  "id": "000006d8c378af1779d2feebc7603a125d99eca0ccf1085959b307f64e5dd358",
  "pubkey": "a48380f4cfcc1ad5378294fcac36439770f9c878dd880ffa94bb74ea54a6f243",
  "created_at": 1651794653,
  "kind": 1,
  "tags": [
    [
      "nonce",
      "776797",
      "20"
    ]
  ],
  "content": "It's just me mining my own business",
  "sig": "284622fc0a3f4f1303455d5175f7ba962a3300d136085b9566801bc2e0699de0c7e31e44c81fb40ad9049173742e904713c3594a1da0fc5d2382a25c11aba977"
}
```

正在验证
----------

下面是一些参考 C 代码，用于计算 NOSTR 注释 ID 中的难度（即前导零位的数量）：

```c
int zero_bits(unsigned char b)
{
        int n = 0;

        if (b == 0)
                return 8;

        while (b >>= 1)
                n++;

        return 7-n;
}

/* find the number of leading zero bits in a hash */
int count_leading_zero_bits(unsigned char *hash)
{
        int bits, total, i;
        for (i = 0, total = 0; i < 32; i++) {
                bits = zero_bits(hash[i]);
                total += bits;
                if (bits != 8)
                        break;
        }
        return total;
}
```

查询电源注释的继电器
-----------------------------

由于中继允许搜索前缀，因此你可以使用此方法来筛选具有一定难度的注释：

```
$ echo '["REQ", "subid", {"ids": ["000000000"]}]'  | websocat wss://some-relay.com | jq -c '.[2]'
{"id":"000000000121637feeb68a06c8fa7abd25774bdedfa9b6ef648386fb3b70c387", ...}
```

委托工作证明
-----------------------

由于 `NIP-01` 笔记 ID 不承诺任何签名，POW 可以外包给 POW 提供商，可能需要付费。这为客户端提供了一种将消息发送到功率受限中继的方法，而无需自己做任何工作，这对于能量受限的设备（如移动设备）非常有用
