---
layout: post
title: "Go panic风险检测"
modified: 2019-04-01 14:53:14 +0800
category: [golang]
tags: [golang,linter]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: 
---

### 背景

最近线上Go项目时不时挂掉，代码质量堪忧。可以写一个代码检测工具，识别panic风险。

### 思路

我们知道：

1. 任意goroutine出现panic，都会导致进程退出。
2. 除了main goroutine（main方法），其他goroutine都是通过go关键字来启动。
3. main goroutine中一般涉及到配置初始化过程，有出现错误及早退出的机制，这里可以不用考虑recover panic。

那么我们就规定，为了防止进程退出，go func()，这个func需要包含recover方法，而且是包含在defer关键字后，居于首位（属于开发最佳实践）。

```
// in a method/function
go sample()

// 
func sample() {
	defer func() {
		if err := recover(); err != nil {
			fmt.Printf("%+v\n", err)
		}
	}()

	// something happen
	panic("hello world")
	// something should happen
}
```


### 使用go/ast

Go自带了一个语法树的包。很多代码检测工具（比如deadcode）都是基于这个做的。

解析出的AST语法树是非常细的。可以检测出go语句，defer语句，如下：

```
// go/ast/ast.go
// A DeferStmt node represents a defer statement.
DeferStmt struct {
	Defer token.Pos // position of "defer" keyword
	Call  *CallExpr
}

// A GoStmt node represents a go statement.
GoStmt struct {
	Go   token.Pos // position of "go" keyword
	Call *CallExpr
}
```

1. 解析得到AST后，取出AST中所有方法node。之后有用。
1. `ast.Inspect`对AST进行深度优先搜索，通过遍历，我们可以查找出所有“go语句”node。
1. 进一步，对`go func()`中的func node检查。从第一步缓存下的方法中取。
1. 检查func node的第一行代码是defer语句，而且包含recover()方法。

POC 在 https://github.com/XUJiahua/panic-risk-detector 。

### TODO

时间原因，简单写到这。很多case没考虑：

1. 支持 go 匿名函数。
1. 支持跨包的检测。
1. 支持外部包的检测，比如http包默认是不处理recover的。

### 参考

* Basic AST Traversal in Go https://zupzup.org/go-ast-traversal/
* 【视频】justforfunc #24: what's the most common identifier in the Go stdlib?
 https://youtu.be/k23xhJoTbI4 利用了scanner.Scanner。词法分析，解析出所有TOKEN。
* 【视频】justforfunc #25: deeper program analysis with go/parser
 https://youtu.be/YRWCa84pykM AST的使用。token真是太基础了，语法树才能激发能量。

https://github.com/yuroyoro/goast-viewer


编译器前端：parser，将文本转换为AST。这是一个苦活，不过Go自带了parser库。这个阶段的一般过程：

1. 词法分析（Lexical analysis）：将文本转换为token（可以使用正则表达式来解析）。
2. 语法分析（Syntactic analysis）：token转化为AST。一些keyword：上下文无关文法，BNF，LL(1) 算法（自顶向下的分析）。

Parser 在编译原理里面是难点但却不是重点，所以在这一部分大家觉得复杂的算法完全可以跳过，不建议浪费太多时间。Parser 都是可以根据正则和 CFG 自动生成的，并不需要自己手写。所以这部分主要目的是学好的是正则和 CFG，那些复杂的算法学起来意义很小。

通过AST静态分析、生成代码。

