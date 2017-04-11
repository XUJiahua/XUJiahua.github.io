---
layout: post
title: "redis dead connection"
modified: 2017-04-09 13:06:58 +0800
category: []
tags: [keepalive, redis, dead connection]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: 
---

## 故障
线上的推送服务不工作了。排除了渠道问题后，定位到使用的队列挤了快60000+的消息没推送。而双活的推送进程一直没挂。

还好打印了日志，从日志上看235应用，4月2日7点07分最后一次从队列获取推送消息；122应用日志，4月2日8点18分最后一次从队列获取推送消息。review了下代码，最可能发生异常的，是从队列获取数据一直是堵塞着。

## 重现

日志说明，两个进程是先后一小时停止服务的，排除了队列服务器的故障。可能就是两个进程所在服务器先后一小时有网络瞬断情况（使用的内网连接）。

队列使用的是redis，怀疑是web server到redis server的网络中断后，连接还在，恢复后这个连接仍不可用。造成堵塞了。本地一试，确实如此。

## dead connection
以下是精简化的代码。queue.DequeuePayload堵塞了，归根结底，是client.Cmd("BRPOP", QUEUE_NAME, TIMEOUT)堵塞了，因为dead connection的原因。

StackOverflow上[这个答案](http://stackoverflow.com/questions/41978922/why-many-libraries-does-not-detect-dead-tcp-connections/41993654)的第二种情况很符合现在的场景。redis堵塞读，请求已经发出去了，客户端一直是在等应答状态，服务端keepalive设置为0，这时候实际连接如果是中断的，那么就会一直阻塞的。


<script src="https://gist.github.com/XUJiahua/a1576b214c3f3f385ccf22379c5f227c.js"></script>

## 解决

使用的redis库没有自动处理dead connection。需要开发者自己解决了。

### 简单解决，加入健康检查：
参考issue，https://github.com/mediocregopher/radix.v2/issues/21

```Go

	// health check
	go func() {
		for  {
			// 确实解决了堵塞不走的问题
			// 观测效果：
			// 堵塞住的dequeue连接恢复了(可能返回EOF或是TIMEOUT)，下一次dequeue就能读取队列消息
			// 有点浪费：
			// 每次PING，其实都在新建连接，有点浪费，还好频率是一分钟一次
			log.Debug("redis connection health check every minute")
			p.Cmd("PING")
			time.Sleep(1 * time.Minute)
		}
	}()
```

TODO:
神奇的事情，没有健康检查，网络瞬断连接还是恢复起来了。而且健康检查根本就帮不上dead connection的问题。
所以说，这个假设是错的？？？推送中断与redis服务器没关系么？？？
现在测试redis server与生产的配置是一样的，特别是timeout, tcp_keepalive。

不可能是channel限流的问题吧？？？

## redis timeout, tcp-keepalive

搜索到这篇[文章](http://rossipedia.com/blog/2014/01/redis-i-like-you-but-youre-crazy/)，总结了下redis server配置对connection的影响。

>The combination of both timeout (0) and tcp-keepalive (0) means that:
> 1. Idle client connections are never killed.
> 1. The server can’t differentiate between an idle client and a dead one.

登录生产redis server查看配置，果然redis server不会主动检测dead connection的。
```
// redis command, get conf of redis server
CONFIG GET *

...
17) "timeout"
18) "0"
...
...
81) "tcp-keepalive"
82) "0"
...
```
