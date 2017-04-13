---
layout: post
title: "移动端数据库Realm及其服务端调研"
modified: 2017-03-29 22:48:43 +0800
category: []
tags: [realm]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: 
---


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

