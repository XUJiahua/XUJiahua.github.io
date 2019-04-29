---
layout: post
title: "rm docker image interactively"
modified: 2019-04-29 17:38:22 +0800
category: []
tags: []
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: 
---


写了个工具交互式删除 docker image（https://github.com/XUJiahua/dockerrmi）。

[demo](https://github.com/XUJiahua/dockerrmi/raw/master/show/show.gif)

实在无法忍受先`docker images`，然后`docker rmi`的重复操作。
还比较原始，刚好够自己目前所需。

想到交互式，是受`ncdu`的启发。

`ncdu`命令是`du`命令的交互式版本。对这种交互式的体验表示好感爆棚，两者体验的差距是巨大的。

![](/media/15565309805917.jpg)


