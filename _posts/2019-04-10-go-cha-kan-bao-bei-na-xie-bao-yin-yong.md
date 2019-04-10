---
layout: post
title: "【Go】查看包被哪些包引用"
modified: 2019-04-10 15:59:33 +0800
category: []
tags: []
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: 
---


做了个工具，https://github.com/XUJiahua/GoPkgDep，查看当前Go包被哪些Go包引用。Goland没找到这类插件。

### 依赖

1. python 3
2. go

### 建立索引

以ethereum源码为例。

```
python3 prePkgDep.py $GOPATH/src/github.com/ethereum/go-ethereum
```

*路径可以是相对路径，绝对路径。*

#### 思路

1. 使用`go list -json`可以获取当前包的imports。
2. 通过递归获取所有包的依赖关系，当前包->被依赖包列表（import）。
3. 建立倒排索引，即被依赖包作为key，被依赖包->包列表。
4. 查询当前包被哪些包引用，直接查倒排索引即可。

### 查询

以ethdb这个包为例。

```
python3 pkgDep.py github.com/ethereum/go-ethereum/ethdb/leveldb
```

### Goland External Tools

配置上述脚本到External Tools，使用上还是挺方便的，对所有Go项目有效。

比如配置prePkgDep.py，主要参数如下：

```
Program: python3
Arguments: /path/to/GoPkgDep/prePkgDep.py $FileDir$
Working directory: $ProjectFileDir$
```

使用：右键项目找external tools，找到上述项目并点击即可。

### 反思

本想通过包依赖图方便自己查看leveldb这个包的引用关系，但结果是一团毛线。

后面想通了，假设依赖关系如下：

```
A <- B
B <- C
```

C与A之间没有必然关系的：假如B包的粒度够粗，C只是引用了B与A无关的那部分代码。

还是得从方法级别查看调用链。


