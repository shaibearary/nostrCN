NIP-22
======

事件 `created_at` 限制
---------------------------

 `draft` `optional` `author:jeffthibault` `author:Giszmo`

继电器可定义其认为事件 `created_at` 可接受的上限和下限。上限和下限必须是中[NIP-01](01.md)定义的 UNIX 时间戳（以秒为单位）。

如果中继支持此 NIP，则中继应向客户端[NIP-20](20.md)发送命令结果，说明由于 `created_at` 时间戳不在允许的限制范围内，因此未存储事件。

客户行为
---------------

客户应使用该[NIP-11](11.md) `supported_nips` 字段了解中继是否使用此 NIP 定义的事件 `created_at` 时间限制。

动机
----------

该 NIP 将对中继接受的事件时间戳的限制正式化，并允许客户端知道具有这些限制的中继。

事件 `created_at` 字段只是一个 UNIX 时间戳，可以设置为过去或将来的时间。继电器接受并共享 20 年前或未来 5 万年的事件。该 NIP 旨在为不希望使用*任何的*时间戳存储事件的中继定义一种方法，以设置它们自己的限制。

如果用户使用错误的系统时钟写入或尝试写入它们，则[可替换事件](16.md#replaceable-events)可能会出现相当意外的行为。现在使用回溯系统持续更新将导致更新不会在没有通知的情况下持续，并且如果他们使用回溯系统进行最后一次更新，他们将再次无法使用现在的正确时间进行另一次更新。

这种 NIP 的广泛采用可以创造更好的用户体验，因为它将减少在遥远的过去或未来出现的大量无序事件，甚至是不可能的日期。

请记住，存在用户将其旧帖子迁移到新中继上的用例。如果中继拒绝不是最近创建的事件，则它不能为此用例提供服务。


Python（伪代码）示例
---------------------------

```python
import time

TIME = int(time.time())
LOWER_LIMIT = TIME - (60 * 60 * 24) # Define lower limit as 1 day into the past
UPPER_LIMIT = TIME + (60 * 15)      # Define upper limit as 15 minutes into the future

if event.created_at not in range(LOWER_LIMIT, UPPER_LIMIT):
  ws.send('["OK", event.id, False, "invalid: the event created_at field is out of the acceptable range (-24h, +15min) for this relay"]')
```
注意：这些只是示例限制，继电器操作员可以选择他们想要的任何限制。
