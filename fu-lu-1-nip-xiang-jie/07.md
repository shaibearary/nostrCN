---
description: 原文：https://github.com/nostr-protocol/nips/blob/master/07.md
---

# NIP07

## Web 浏览器的 `window.nostr` 功能

`draft` `optional` `author:fiatjaf`

该 `window.nostr` 对象可以通过网络浏览器或扩展变得可用，并且网站或网络应用程序可以在检查其可用性之后使用它。

该对象必须定义以下方法：

```
async window.nostr.getPublicKey(): string // returns a public key as hex
async window.nostr.signEvent(event: Event): Event // takes an event object, adds `id`, `pubkey` and `sig` and returns it
```

除了上述两个基本功能外，还可以选择实现以下功能：

```
async window.nostr.getRelays(): { [url: string]: {read: boolean, write: boolean} } // returns a basic map of relay urls to relay policies
async window.nostr.nip04.encrypt(pubkey, plaintext): string // returns ciphertext and iv as specified in nip-04
async window.nostr.nip04.decrypt(pubkey, ciphertext): string // takes ciphertext and iv as specified in nip-04
```

### 实施

* [nos2x](https://github.com/fiatjaf/nos2x)（Chrome 及其衍生产品）
* [Alby](https://getalby.com)（Chrome 及其衍生产品、Firefox、Safari）
* [Blockcore](https://www.blockcore.net/wallet)（Chrome 及其衍生产品）
* [nos2x-fox](https://diegogurpegui.com/nos2x-fox/)（Firefox）
* [Flamingo](https://www.getflamingo.org/)（Chrome 及其衍生产品）
