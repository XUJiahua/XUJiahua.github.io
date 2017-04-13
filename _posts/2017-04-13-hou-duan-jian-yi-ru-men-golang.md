---
layout: post
title: "移动端转后端简易入门（Golang）"
modified: 2017-04-13 09:07:44 +0800
category: []
tags: [Golang, Tutorial]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: 
---

简易入门从写RESTful接口开始，与移动开发关系最近。Golang的入门可以从官网开始，也可以看下Golang仓库[Wiki](https://github.com/golang/go/wiki#getting-started-with-go)。完完整整看一遍，看不懂没关系，以后看。这里大概介绍下要点，作为思路。

### 开发工具
最好选用Linux/Mac平台开发。编辑器使用IntelliJ IDEA，加上Golang插件。个人认为不要把太多精力放在琐碎事情上，重点是Golang开发，选择最方便的，开箱即用型的。

### $GOPATH
Golang项目一定要在$GOPATH目录下。
1. 可以考虑把所有Golang项目都放在一个$GOPATH路径下
1. 也可以为每个Golang项目建一个$GOPATH，开发前 export GOPATH=/some/path
1. 或者为$GOPATH路径外建立的项目建立一个到$GOPATH的软连接 ln -s \`pwd\` $GOPATH/some/path


### HTTP协议

RESTful接口是基于HTTP协议上的。

返回码2xx,3xx,4xx,5xx都代表什么意思？GET，POST有什么区别？一些典型的HTTP Header代表什么意思？HTTP协议是否有状态，cookie/session的作用是什么？不清楚的，找些科普文了解下。

现在常用的协议版本是HTTP/1.1，HTTP/2.0又有什么样的改进，也需要了解下，很快就会普及了。

### RESTful接口设计

推荐几篇文章：

1. [撰写合格的REST API](http://mp.weixin.qq.com/s?__biz=MzA3NDM0ODQwMw==&mid=208060670&idx=1&sn=ce67b8896985e8448137052b338093e0) 满足合格，不容易的。
1. [Google API Design Guide](https://cloud.google.com/apis/design/) 关于RESTful，gRPC的接口设计


### 路由

Angular/Vue/React等前端框架库流行的今天，前后端分离已经是一种通用的开发解耦手段，前后端通过接口约定来合作。服务端渲染页面的方式不大常用了。服务端只要写接口就行了。

可以看看这篇文章[Web 研发模式演变](https://github.com/lifesinger/blog/issues/184)，看看Web开发的演变过程。

故，没必要选一些很重的Web库。

Golang自带http server，不像Java war包放到容器才能运行。自带的路由库足够用了，但还是比较原始。

这里推荐用[gorilla](http://www.gorillatoolkit.org/)，tutorial[见此](https://www.thepolyglotdeveloper.com/2016/07/create-a-simple-restful-api-with-golang/)。


### 数据库
https://github.com/golang/go/wiki/SQLDrivers

### 日志

排查问题，日志是非常有效的线索。

### panic & recover

注意任何goroutine中的panic都会导致整个进程中断。即使没有主动panic，要是代码写出数组越界这样的错误，也会引发panic。最稳妥的办法就是每个goroutine做好recover工作，简单来说，在开始前加入如下代码：

```Go
defer func() {
	if e := recover(); e != nil {
    // recover from an error
	}
}()
```

### goroutine并发（线程并发、数据同步）

几个goroutine并发读写共享变量，一定要保证顺序访问变量，特别是写的时候。Golang提供了两种方式：channel和sync包。

### go test（单元测试）

### 接口测试

1. 用Postman模拟请求
2. 通过端口转发的方式，实时将测试环境的流量导入到本地开发环境，让调用方帮你测（懒人做法）

### package init

>If a package p imports package q, the completion of q's init functions happens before the start of any of p's.
>The start of the function main.main happens after all init functions have finished.

package的初始化工作都可以放在init方法中，所有package的init方法都会在调用main方法前执行。

### 包管理
Golang的包（第三方包）管理还比较稚嫩，相比Java的Maven/Gradle。

笔者用的是[godep](https://github.com/tools/godep)。
```
// installation
go get github.com/tools/godep

// This will save a list of dependencies to the file Godeps/Godeps.json and copy their source code into vendor/ (or Godeps/_workspace/ when using older versions of Go)
godep save

// 建议将生成出的vendor也放入版本管理中（Golang找package的时候优先从本地vendor目录找，其次$GOPATH下，再次$GOROOT）
```

其他Golang包管理工具见：
https://github.com/golang/go/wiki/PackageManagementTools

### 开发规范
这方面可以参考下V2Ray的开发规范：
https://www.v2ray.com/chapter_04/01_guide.html

注意code review。

### Linux

Linux常用命令/Shell脚本
```
网络 netstat
磁盘 df du
内存 free
CPU ps htop
```

### Nginx

反向代理服务器，作为负载均衡器。

### 性能测试

性能测试另起章节再说。不得不说，业务提需求的时候最好也要写好需要支持的并发量，作为测试验收的一个环节，强制执行。

### 自动化部署 CI/CD

利用Jenkins, Ansible等工具，提高部署速度。

### 服务器／服务监控

日志收集，开源软件ELK（ElasticSearch, Logstash, Kibana）。具体说来，Logstash安装在应用服务器，将应用日志发送到ElasticSearch服务器上，通过Kibana来分析日志和查看日志。


### 参考

1. Web 研发模式演变 https://github.com/lifesinger/blog/issues/184
1. 撰写合格的REST API http://mp.weixin.qq.com/s?__biz=MzA3NDM0ODQwMw==&mid=208060670&idx=1&sn=ce67b8896985e8448137052b338093e0
1. golang gorilla http://www.gorillatoolkit.org/
1. godep https://github.com/tools/godep
1. awesome go https://github.com/avelino/awesome-go