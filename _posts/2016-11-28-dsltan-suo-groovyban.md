---
layout: post
title: "DSL探索(Groovy版)"
modified: 2017-04-13 17:32:20 +0800
category: []
tags: []
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: 
---

DSL( domain-specific language )，具体领域语言，姑且这么翻译。
DSL的作用是什么呢，以我粗浅的理解，就是配置文件的描述语言。

具体来说，分为external DSL、internal DSL。
像XML、JSON的配置文件，也被划分为external DSL。
那些在比如Ruby、Groovy上建立的DSL，被称为internal DSL，或是embedding DSL。
从表现形式来看，internal DSL也更强大，配置即代码。

## 使用场景举例

笔者工作中碰到过几个场景：
- iOS开发中会使用CocoaPods作为第三方包管理工具，podfile其实是基于Ruby的DSL
- Jenkins Pipeline中会使用基于Groovy的DSL（embedding DSL的隐藏好处就是，你可以写代码，不一定需要拘泥于DSL语法了）
- Android Gradle脚本，其实用的是Groovy的DSL
- TODO：MongoDB的查询语法其实也是一种DSL，我们可以用Groovy设计下 http://www.jroller.com/DhavalDalal/entry/a_case_for_using_groovy

## Groovy语言

设计一个DSL(embedding DSL)，目的是让使用者能够简洁得表达一件事情。Java语言太啰嗦，Groovy作为“新”语言，表达力非常棒，可以上官网研读。

1. Groovy closure：比起Java，Groovy是真简洁，很多类都用一个高阶函数/closure替换就行。（Objective-C Block, JavaScript function）
2. Groovy可以接受没有括号（）的传参数，这点对DSL来说很有用
3. data is code, 模糊了data与code的边界，使用GroovyShell来解析dsl（JS/Ruby eval）
4. MetaProgramming的能力，静态元编程，AST抽象语法树的操作。


## DSL与核心代码怎么整合

TODO

DSL驱动的开发模式，感觉都是围绕Java为核心做的。
因为Java与Groovy结合的比较紧密，对golang来说就不是很适用了。


## Demo: Groovy版CocoaPods

## References:
1. http://docs.groovy-lang.org/latest/html/documentation/core-domain-specific-languages.html
1. http://groovy-lang.org/metaprogramming.html
1. book: DSLs in Action （主要是Java系的，2010年的书，可能语法上有些不同了）