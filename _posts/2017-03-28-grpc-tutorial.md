---
layout: post
title: "服务端开发之接口规范"
modified: 2017-03-28 17:38:48 +0800
category: [服务化]
tags: [gRPC,RPC,SOAP,8583]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: 
---

### SOAP->RESTful->gRPC

#### 一 SOAP
第一家公司使用的技术栈是.NET/Java，用的接口规范是SOAP（Simple Object Access Protocol），一种比较老的RPC协议，开发完代码，自动生成接口规范，调用方直接根据规范生成客户端代码，.NET程序与Java程序也比较方便交互。

#### 二 RESTful
移动开发的崛起，这种基于HTTP、数据格式是XML的规范越发不受待见了，在数据传输量上，XML相比JSON笨重。进入第二家公司，周围的人都不用SOAP规范的，取而代之的是基于HTTP/JSON的RESTful接口。但是调试RESTful接口有几件令人痛苦的事情：

1. 没有规范文档，很尴尬
2. 客户端代码还要自己写，很尴尬

这种场景下，调试RESTful接口令人心烦啊，各种求人。如何解决呢？

1. 没文档，使用[Swagger](http://swagger.io/)生成API文档，还是需要点工作量的
2. 客户端代码，还是得自己写，用一些好用的客户端框架比如OkHttp

第二家公司还有使用[8583协议](https://en.wikipedia.org/wiki/ISO_8583)的支付接口，在扩展性和调试（有学习曲线，熟练之后调试就不是主要问题了）方面，都非常差。

#### 三 gRPC
兼顾传输效率、开发效率，还是得选用下现代的RPC（Remote Procedure Call）框架。
报文序列化、构造网络请求、服务端路由都是RPC框架替开发者完成的。

开发者需要做的：
1. 客户端调用方法
2. 服务端实现接口逻辑

调试接口是很麻烦的事情，让RPC框架做这些事情吧，效率点，早点收工回家！用RPC生成各个语言的客户端也是很方便的，比如Evernote使用thrift作为SDK与服务端的通讯桥梁。

每个RPC框架基本就是在现有基础上的封装：
1. 选择一种传输协议HTTP/HTTP2/TCP
2. 选择一种传输格式JSON/XML/Protobuf

gRPC是基于HTTP2/Protobuf的RPC框架。在数据传输效率上是非常好的，支持的开发语言也很多，Android/iOS/Java/Golang/Python，主流开发语言都支持的。

注意一点，Nginx转发HTTP2流量只能是在四层转发。

> Nginx today doesn't support HTTP2 as upstream, so you cannot use it as a L7 proxy for GRPC. Instead you can use it as a L4 proxy.
>
For the RST_STREAM issue, it's a known regression in nginx.
https://trac.nginx.org/nginx/ticket/959
>
There are no plans to implement HTTP/2 support in the proxy module in the foreseeable future, see​detailed answer here. If you want to use nginx to balance multiple servers, consider using ​the stream module to do this.
https://trac.nginx.org/nginx/ticket/923


### 基于gRPC的iOS前端与Golang后端的通信

#### 1. 安装gRPC类库、插件

```
// 安装grpc类库和ProtoBuffer的插件，会自动安装ProtoBuffer（版本为3.0.2）
brew install --with-plugins grpc
// 确认下插件是否已经安装
which protoc-gen-objcgrpc

```

备注：Java source code is in the grpc-java repository. Go source code is in the grpc-go repository.

#### 2. 生成代码
```
protoc -I polling/ polling/polling.proto --go_out=plugins=grpc:polling --objc_out=polling --objcgrpc_out=polling
```

生成的代码copy到iOS工程。
这里生成的go代码是给服务端用的。

#### 3. iOS
podfile中添加依赖：

```
pod 'Protobuf', '~> 3.1.0'
pod 'gRPC-ProtoRPC', '~> 1.0.1-pre1'
```

#### 4. 禁用TLS，如果服务端没有证书
```
#import <GRPCClient/GRPCCall+Tests.h>

[GRPCCalluseInsecureConnectionsForHost:@"127.0.0.1:10000"];
```

>
Notice that before constructing our service object we’ve told the gRPC library to use insecure connections for that host:port pair. This is because the server we will be using to test our client doesn’t use TLS. This is fine because it will be running locally on our development machine. The most common case, though, is connecting with a gRPC server on the internet, running gRPC over TLS. For that case, the useInsecureConnectionsForHost: call isn’t needed, and the port defaults to 443 if absent.



#### 5.参考：
* http://www.grpc.io/docs/tutorials/basic/objective-c.html
* https://github.com/grpc/homebrew-grpc
* http://www.grpc.io/docs/quickstart/objective-c.html#prerequisites
