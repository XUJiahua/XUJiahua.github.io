---
layout: post
title: ".NET多线程小结"
modified: 2014-03-10 22:43:32 +0800
category: []
tags: [多线程]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: 
---


工作内容是支付相关的，整天就是集成银行、第三方支付接口。工作，每天就是重复，人也比较疲惫。要么辞职不干，要么打上一针鸡血。

鸡血来了~

QA做regression testing的时候用力太猛，短短几个小时，系统爆了，error邮件6W+。基本就一个错，“MSMQ没有足够资源了”。排查下来，这个错只是表象，深层原因还是queue consumer不给力，是单线程的。可以想象一下，10个厨师每人正以每秒一道的速度为你做菜，持续了一个小时，桌子再大也不够用的啊。倒是PRO上从没出过问题，可以想见这个系统的用户量有多小。

多线程编程挺令人期待的呢，兴奋。着手demo。

第一版：利用MSMQ实例方法beginreceive是线程安全的特性，需要几个工作线程就call几次beginreceive方法。为保持工作线程不断，拿到message处理完，继续调用beginreceive方法。因为这个queue consumer是windows service，还得处理stop事件。竟然使用了while(counter!=0)这样的方式来等待工作线程结束，busy waiting耗费CPU。起飞挺不错的，降落不是一般差。。。

（不过神奇的是，这个方案被枪毙的原因是.NET高级版本不保证beginreceive线程安全。不升级就好了嘛！！！）

-- 这版里一点多线程的概念都没有，自己也不满意，啥都没学到呢。

第n版：既然连让我完美起飞的beginreceive都不能放心用了，只能换种方式。由调用beginreceive开启的线程当作dispatcher线程，用于取message再分发到worker pool去处理。现在的问题是得控制worker pool的size，最多让n个worker thread运行，还得考虑stop事件，得等所有worker thread运行完才能准程序退出。这次不能用busy waiting！查了MSDN，看了所有用于线程同步的工具，最终确定使用semaphore，用来限制多个线程并发访问资源。

（这次review直接让审查者看不明白了，50多岁了第一次知道semaphore这东西。review之后就没回音了。）

最终版：TBD。。。设计健壮程序，一定要考虑边界条件，极端条件。这部分就与多线程编程关系不是很大了。比如一个潜在问题：windows service stop事件会否出现timeout，特别是处理message时间过长，比如连接sql server timeout了。深层次的原因是，没动力了。拥有n年工作经验、C++写了30w行的新同事建议把所有queue consumer整合在一起，提高扩展性，至于稳定性可以再想办法，感觉找到知音了。向上提，直接被reject，因为现在程序跑得好好的。其实对于喜欢稳的上级，你得把东西做好了再往上提的。这么忙，想想也就算了。

扎下这针鸡血快一个月了，最近又进入疲惫期 - -

ps: 为不误人子弟，建议直接看好书（如[C#线程参考手册](http://book.douban.com/subject/1141299/)）或是MSDN。