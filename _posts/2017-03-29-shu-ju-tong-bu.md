---
layout: post
title: "数据同步"
modified: 2017-03-29 22:23:02 +0800
category: []
tags: [realm,offline,online,数据同步]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: 
---

--

终端做联机交易，查账单、交易统计及结算数据也是查了服务端。联机交易无可厚非。但是账单、统计其实可以做成本地的，一是减少了网络连接，减小服务器压力，二是离线状态下，也能方便得查看今日收款，查查某笔订单的详情。想想本地存储好处也是很多的呢。

本地存一份，服务器也有一份，数据肯定是有不一致的地方，本地交易数据的状态与服务端保持一致，这份账单才是有意义的。

这个同步冲突模型，对我们的服务需要改改：

1. 服务端／客户端都没有删除操作。
1. 服务端有插入、更新操作
1. 同步永远是服务端->客户端。
1. 客户端只有插入操作，没有更新操作
1. 客户端更新操作，只有一种情况，服务端数据更新了。即，同步。
1. 以服务器最新的时间戳为准。
1. TODO：需要细化好多东西

借鉴了下Realm的冲突解决策略

```
Conflict Resolution
One of the defining features of mobile is the fact that you can never count on being online. Loss of connectivity is a fact of life, and so are slow networks and choppy connections. But people still expect their apps to work!

This means that you may end up having two or more users making changes to the same piece of data independently, creating conflicts. Note that this can happen even with perfect connectivity as the latency of communicating between the phone and the server may be slow enough that they can end up creating conflicting changes at the same time.

What Realm does in this case is that it merges the changes after the application of specific rules that ensure that both sides always end up converging to the same result even, though they may have applied the changes in different order.

This means that you no longer have the kind of perfect consistency that you could have in a traditional database, what you have now is rather what is termed “strong eventual consistency”. The tradeoff is that you have to be aware of the rules to ensure the consistent result you want, but the upside is that by following a few rules you can have devices working entirely offline and still converging on meaningful results when they meet.

At a very high level the rules are as follows:

* Deletes always wins. If one side deletes an object it will always stay deleted, even if the other side has made changes to it later on.

* Last update wins. If two sides has updated the same property, the value will end up as the last updated.

* Inserts in lists are ordered by time. If two items are inserted at the same position, the item that was inserted first will end up before the other item. This means that if both sides append items to the end of a list they will end up in order of insertion time.
```

ref: https://realm.io/docs/realm-object-server/#conflict-resolution

