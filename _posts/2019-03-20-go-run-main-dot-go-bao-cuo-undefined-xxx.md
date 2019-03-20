---
layout: post
title: "go run main.go 报错 undefined xxx"
modified: 2019-03-20 20:52:11 +0800
category: [golang]
tags: [golang]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: 
---

一般，出现这个问题，是因为main包中有多个源文件。比如以太坊的geth包，github.com/ethereum/go-ethereum/cmd/geth。

解决办法就是，让 go run 知道所有的源文件：

```
go run `ls *.go | grep -v "test"`
```

翻看`go help run`，发现还能更简单：

```
go run . 
```

dlv debug 面对main包中有多个文件的情况，也有类似的问题，解决方式同上。

同样的报错，使用Goland的Debug时也遇上过。

解决方案如图，Run kind改为Directory，Directory选择main.go所在的目录。

![](/media/15530864500054.png)


