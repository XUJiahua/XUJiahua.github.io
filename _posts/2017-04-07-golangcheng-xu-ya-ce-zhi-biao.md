---
layout: post
title: "Golang程序压测指标"
modified: 2017-04-07 13:28:48 +0800
category: [Golang]
tags: [Golang, 压力测试]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: 
---

## 背景
之前版本的APP消息推送服务做得保守了，考虑到同一台主机上还有联机交易系统还有提供下载报表的平台，同时处理消息推送的goroutine数量设置在10条。一直运行得稳稳的，直到交易量上来了。用户已经能明显感觉到推送的延迟了（用户场景：顾客扫完固定码做支付宝、微信交易后，交易成功的消息将推送到商户手机上）。

## 观测
程序中加入了到渠道的推送占用时间后，观测到：
1. Android推送（使用友盟和小米）占用时间20ms~250ms；
1. iOS推送（APNS）占用时间5s~6s；
1. iOS推送虽然需要5s~6s，实际感受是手机先收到推送，服务器后面再收到推送返回结果；相比下来，Android推送渠道对手机推送的处理应该是异步处理的。


## 推理
iOS推送明显太慢了，如果同时要推10台iOS设备，就把10条goroutine占满了，其他消息就得等到5s后继续处理了，对实时性影响很大。

不限制并发数，怕是流量异常把服务器撑坏了，那就提高下并发推消息的goroutine最大数吧。

在测试环境分配给推送用的goroutine数为最大1000。模拟了10000笔／秒的交易量，响应速度很好。

stackoverflow上（http://stackoverflow.com/questions/8509152/max-number-of-goroutines
）也有指出goroutine的开销很小，每个goroutine占用2k内存，GC会稍慢些。这样1000个goroutine不算多。

## 通过工具对比下程序的反应 10x goroutine vs. 1000x goroutine

监控指标一般如下：
1. CPU
1. 内存
1. goroutine使用数量
1. GC暂停
1. 文件打开数（未找到好的工具）
1. TCP连接数（未找到好的工具）
1. QPS
1. 响应时间

搜索了下资料，Golang上的监控工具有这些。

### debugcharts
https://github.com/mkevac/debugcharts

可视化监控CPU使用情况、内存分配、GC暂停。需要修改代码，添加一个http handler，需要建http服务。

```
// installation
go get -v -u github.com/mkevac/debugcharts

// add a http end-point
import _ "github.com/mkevac/debugcharts"

```
CPU使用率（10 goroutines）
![CPU 10](/images/golang/CPU_10.png)
CPU使用率（1000 goroutines）
![CPU 1000](/images/golang/CPU_1000.png)
内存使用率（10 goroutines）
![MEM 10](/images/golang/MEM_10.png)
内存使用率（1000 goroutines）
![MEM 1000](/images/golang/MEM_1000.png)
GC暂停时间（10 goroutines）
![GC 10](/images/golang/GC_10.png)
GC暂停时间（1000 goroutines）
![GC 1000](/images/golang/GC_1000.png)

同样的交易量，1000 goroutines 10s就处理完了所有消息推送，而10 goroutines 需要7分钟。简单分析几个指标：
1. 1000 goroutines占用的CPU也不多。
1. 内存占用看不出区别
1. 注意，1 second = 1e+9 nanoseconds，图中GC暂停最多是240k nanoseconds = 0.00024s。

### gom
https://github.com/rakyll/gom

监控goroutine数量，thread数量，以及CPU、内存占用最多的代码段。（可能是更新了golang的原因，CPU、内存占用的profile gom解析不了了）

```
// installation
go get github.com/rakyll/gom/cmd/gom

// add a http end-point
import _ "github.com/rakyll/gom/http"

```

### gcvis
https://github.com/davecheney/gcvis

gctrace的实时可视化，不修改代码

```
gcvis ./app

```

### golang本身

如果开启了http服务，golang默认提供了一个接口，显示memstats信息
http://localhost:7000/debug/vars

runtime包中也能读取很多有用信息。

## 反思

这个服务做的匆忙，也没测出瓶颈就上线了。做好服务的一个前提，就是要规范化，规范要求服务上线前做好压力测试。并发量、稳定性都是量大的时候容易出现的瓶颈。服务的代码逻辑是最没难度的。