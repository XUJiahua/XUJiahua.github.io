---
layout: post
title: "PayPal Integration"
modified: 2015-03-19 07:31:36 -0800
category: []
tags: [Payment,产品,PayPal]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: 
---

## PayPal Checkout ##

身处支付平台项目，最近添加了类似PayPal的支付页面。自觉设计得不够好，主要是有安全问题。本文会介绍PayPal ExpressCheckOut的流程，好在哪。

从用户角度，整个支付流程是这样的：

![](http://xujiahua.github.io/images/paypal/paypalcheckoutflow.png)


1. 购物车页面，PayPal Checkout
![](http://xujiahua.github.io/images/paypal/cart.png)

1. 这一步是隐藏步骤，用户不可见。PayPal Checkout按钮并没有直接将用户引导到PayPal，而是先去了商家后台，商家后台call PayPal API SetEC，验证商家（**在后台传输credential总比在浏览器端传输安全，这也是我所说内部项目的安全问题**）并在PayPal创建一个订单，会有一个token返回，商家后台跳转，引导用户到PayPal。
![](http://xujiahua.github.io/images/paypal/paypalsetec.png)
1. 用户被跳转到sandbox页面，登录、选择支付方式、选择收货地址。
![](http://xujiahua.github.io/images/paypal/paypalsandbox.png)
1. 用户选择continue后，PayPal将页面跳转到商家预定义的returnurl上。
![](http://xujiahua.github.io/images/paypal/paypalsandbox2.png) 
1. 本页面就是商家预定义在PayPal的returnurl。控制权又交给商家/用户，在这里通过token call PayPal API GetEC取出订单信息（optional），并让用户确认订单，call PayPal API DoEC。
![](http://xujiahua.github.io/images/paypal/paypalgetec.png)
1. 支付结果。
![](http://xujiahua.github.io/images/paypal/paypaldoec.png)

说得比较简略，可以从[这里](https://demo.paypal.com/navigation?device=desktop&merchant=beauty)得到demo的代码。

## 后续流程还有settlement ##

很好奇DoEC之后为何还有DoAuth、DoCapture的API。为此看了下PayPal API描述，摘录如下：

> A sale is the most straightforward payment action. Choose this payment action if the transaction, including shipping of goods, can be completed immediately. To use this payment action:
> 
> The final amount of the payment must be known when you invoke the DoExpressCheckoutPayment API operation
> You should intend to fulfill the order immediately, such as would be the case for digital goods or for items you have in stock for immediate shipment
> 
> After you execute the DoExpressCheckoutPayment API operation, the payment is complete and further action is unnecessary. You cannot capture a further payment or void any part of the payment when you use this payment action.
> 
> An authorization payment action represents an agreement to pay and places the buyer's funds on hold for up to three days.
> Choose this payment action if you need to ship the goods before capturing the payment or if there is some reason not to accept the payment immediately.
> 

其实主要问题就是发货。国外买诸如电脑之类的实物，往往是信用卡上先冻结，货物收到才从商家那扣款的。