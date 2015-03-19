---
layout: post
title: "京东支付流程之微信支付"
modified: 2015-03-19 07:31:36 -0800
category: []
tags: [Payment,产品]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: 
---

# 京东支付流程之微信支付 #
*去年10月记在Evernote里的笔记，现在稍作整理。*

京东系统分成多个子模块，比如分为支付、购物车等。系统模块化在大公司很常见，每块都有独立管理，维护好对外接口即可。

## 购物车模块 cart.jd.com ##
把产品加入购物车后：

![](http://xujiahua.github.io/images/payment/cart.jd.com.png)

## 交易模块：trade.jd.com ##
选择收货地址与支付方式：

![](http://xujiahua.github.io/images/payment/trade.jd.com.png)

## 支付模块：pay.jd.com ##

选择支付平台，可以选择具体银行：

![](http://xujiahua.github.io/images/payment/pay.jd.com.png)
也可以选择第三方支付平台。这里，我们选择微信支付：

![](http://xujiahua.github.io/images/payment/pay.jd.com2.png)

JD没有使用长连接技术，还是做客户端轮询：

![](http://xujiahua.github.io/images/payment/pay.jd.com3.png)
一旦AJAX请求得到支付更新状态，页面跳转并提示成功：
![](http://xujiahua.github.io/images/payment/pay.jd.com4.png)

这里总结下整个流程，上文没有说到微信客户端，这里会带过：

1. 浏览器（客户端）请求后台，生成二维码页面。并通知微信支付后台，微信支付后台push支付请求到微信上。
1. 浏览器（客户端）定期请求（轮询）服务器（这里是腾讯微信支付后台），请求支付状态。
1. 微信（客户端）上支付成功后，支付状态存储到了微信支付后台。
1. 浏览器在轮询中，发现支付状态更新，浏览器页面更新。
1. 支付结束。

同理，支付宝的穿越到手机支付也类似。

在环节2中，JD采取的是client pull的方式，客户端定期请求服务端。与之对应的技术是server push，由服务器来控制消息更新的节奏。可以尝试用下Python Tornado下。之所以server push无法普及，与老版本浏览器兼容问题有关。
这块待续。


