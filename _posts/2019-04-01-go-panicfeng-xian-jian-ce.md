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

