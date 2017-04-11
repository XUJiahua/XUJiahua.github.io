---
layout: post
title: "从一次Golang服务优化，学习下压测指标"
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
之前版本的APP消息推送服务做得保守了，考虑到同一台主机上还部署了联机交易系统和提供下载报表的平台，同时处理消息推送的goroutine数量设置在10条。一直运行得稳稳的，直到交易量上来了。用户已经能明显感觉到推送的延迟了。

用户场景：顾客扫完固定码做支付宝、微信交易后，交易成功的消息将推送到商户手机上。

## 观测
代码中加了日志，统计了到渠道的推送占用时间后，观测到：
1. Android推送（使用友盟和小米）占用时间20ms~250ms；
1. iOS推送（APNS）占用时间5s~6s；
1. iOS推送虽然需要5s~6s，实际感受是手机先收到推送，服务器后面再收到推送返回结果；相比下来，Android推送渠道对手机推送的处理应该是异步处理的。


## 推理
iOS推送明显太慢了，如果同时要推10台iOS设备，就把10条goroutine占满了，其他消息就得等到5s后继续处理了，对实时性影响很大。

不限制并发数，怕是流量异常把服务器撑坏了，那就提高下并发推消息的goroutine最大数吧。

在测试环境分配给推送用的goroutine数为最大1000。模拟了10000笔／秒的交易量，响应速度很好。看看两者在系统资源上的占用对比吧。

## 10x goroutine vs. 1000x goroutine

监控指标一般如下：
1. CPU
1. 内存
1. goroutine使用数量
1. GC暂停
1. 文件打开数（未找到好的工具）
1. TCP连接数（未找到好的工具）
1. QPS
1. 响应时间


通过工具对比下程序的反应 
stackoverflow上（http://stackoverflow.com/questions/8509152/max-number-of-goroutines
）也有指出goroutine的开销很小，每个goroutine占用2k内存，GC会稍慢些。这样1000个goroutine不算多。


搜索了下资料，Golang上的监控工具有这些。

### pprof

Golang内置的工具包。

```
// graphviz是开源的
brew install graphviz

// 引入http handler
import _ "net/http/pprof"

// look at a 30-second CPU profile
// 生成系统调用图
go tool pprof -svg http://localhost:7000/debug/pprof/profile > pprof001.svg
```

这里svg图显示不全，请在新标签打开图像。可以看到占用CPU最多的方法。这里因为是推送，主要是APNS连接占用的CPU比较多，而且是TLS握手的占用资源多，可能用新APNS接口会好些。还可以采样heap profile等。

![pprof report](/images/golang/pprof001.svg)

之后的工具都是对pprof的封装，提供了更好的可视化效果。主要是增加了时间维度后，看趋势图更直观。

### go-torch 火焰图，CPU占用监控

```
// installation
go get github.com/uber/go-torch
git clone git@github.com:brendangregg/FlameGraph.git
export PATH=$PATH:/path/to/FlameGraph

// generate
go-torch --file "torch.svg" --url http://localhost:7000
```

效果如下，与pprof的显示方式不同而已。
![torch report](/images/golang/torch.svg)


### [debugcharts](https://github.com/mkevac/debugcharts) Web实时显示

可视化监控CPU使用、内存分配、GC暂停。需要修改代码。但没有goroutine数量的统计。

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
1. 注意，1 second = 1e+9 nanoseconds，图中GC暂停最多是240k nanoseconds = 0.00024s。这点时间，基本可忽略不计了。


在测试环境使用htop观测CPU占用率，1000 goroutine火力全开，最多占用了200%CPU（四核系统，相当于占用了两核），而本地Mac htop/top显示都没有压力，图也是在Mac上截取的。Mac上htop监控不到我程序的CPU和内存使用啊，所有指标都是0..
看来需要租一台云主机来处理了。

这里的CPU usage可以理解为CPU usage per second。这里占用了0.8，已经不得了了。4核心，单位时间提供4s的CPU时间，0.8相当于占用了一核。

TODO：
socket
file descriptor
应该都是server级别的监控啊。


### [gom](https://github.com/rakyll/gom) 终端实时显示

监控goroutine数量，thread数量，以及CPU、内存占用最多的代码段。需要修改代码。（TODO：可能是更新了golang的原因，CPU、内存占用的profile gom解析不了了）
gom比较直观，goroutine、thread的数量展示，容易查看哪块代码的cpu、heap占用较高，按照使用率从高往低排序
对一个能用的程序优化，使用gom最简洁，就看cpu、heap占用率，解决高峰值的那几个代码块。


```
// installation
go get github.com/rakyll/gom/cmd/gom

// add a http end-point
import _ "github.com/rakyll/gom/http"

```

### [gcvis](https://github.com/davecheney/gcvis) 监控Golang的垃圾回收

gctrace的实时可视化，不修改代码。

```
gcvis ./app

```

## 反思

做好服务的一个前提，就是要规范化，规范要求服务上线前做好压力测试，知道瓶颈。服务好坏的一个标准就是，并发量上来，还是稳定。服务的代码逻辑是最没难度的。


## 参考

1. A Short Survey of PProf Visualization Tools http://slcjordan.github.io/post/pprofsurvey/
1. pprof package https://golang.org/pkg/net/http/pprof/
1. pprof detailed end-user documentation https://github.com/google/pprof/blob/master/doc/pprof.md
1. http://holys.im/2016/12/13/when-we-benchmark-what-should-we-care/index.html