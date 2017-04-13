---
layout: post
title: "Evernote的增量更新机制"
modified: 2017-04-12 15:25:41 +0800
category: []
tags: [Evernote, Update Sequence Number, 增量更新]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: 
---

Evernote的同步设计见此：https://dev.evernote.com/media/pdf/edam-sync.pdf

### Update Sequence Number
整个同步的基石就是Update Sequence Number。

每个数据元素比如Evernote的每条笔记，都有一个字段USN(Update Sequence Number)。每个账号都维护了一个USN，USN从1开始（新增第一个元素时是1），任何数据元素的创建、修改、删除操作，USN+1，并记录到数据元素的USN字段。

USN字段需要加上索引，因为需要根据这个字段排序。


### 增量更新
客户端上次同步成功时的最大USN，记为last update count，请求增量数据的时候需要带上这个字段。增量同步，就是将所有USN大于update count的数据元素下发到客户端。数据量大的话，做好分页（还是以USN作为分页参数）。

全量更新，就是last update count是0的情况。


### USN会不会爆炸
以一个4字节（32bit）的Integer为例，2^32 = 4294967296，可以支持42亿次的更新。能爆炸的可能性不大。建议用下8字节的长整型，避免意外。


### 终端本地账单的数据同步

Evernote这种同步方式挺省流量的，只有改动的数据会做同步。打算采用这个方式应用到业务上。

现在有一个交易库，要满足POS端从服务端以Evernote的形式增量获取交易数据，需要满足如下条件：
1. 交易表加一个USN(Update Sequence Number)字段，64bit Integer数值类型（老数据默认是0）。
2. 每个终端（商户号+终端号）都维护一个USN（从0开始）。每次更新（包括插入）交易数据，USN+1，并更新到交易记录中。
3. 上线后运行30天，保证30天（POS端只留30天数据）内的数据都有USN。


最终的数据流这样：
1. POS发起交易，交易数据不用存数据库
1. 触发了同步后，交易数据从服务端增量拉取，并存本地
1. 此时，POS端、服务端数据保持一致。本地数据其实就是服务端数据的copy
1. 交易统计、交易明细、交易结算的数据源就是本地数据

本地账单的好处，减少了网络连接次数，帮服务端挡了压力。对用户来说，弱网环境也能查交易，感觉良好。

