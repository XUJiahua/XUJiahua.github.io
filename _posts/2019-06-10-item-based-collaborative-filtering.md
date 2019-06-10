---
layout: post
title: "item-based collaborative filtering"
modified: 2019-06-10 11:43:48 +0800
category: []
tags: []
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: 
---

本文的基础是下面这篇文章，本文主要着重讲几个容易忽视的问题。

Item-item collaborative filtering with binary or unary data
https://medium.com/radon-dev/item-item-collaborative-filtering-with-binary-or-unary-data-e8f0b465b2c3


### item-item similarity matrix

![](/media/15601386053473.jpg)

### cosine similarity and normalization

看看没有归一化（normalization）有什么问题。

假设：

1. 用户A对商品的评分：(1, 1, 1, 1, 1)
1. 用户B对商品的评分：(5, 5, 5, 5, 5)

可知，两个用户的品味是不同的。

而根据余弦相似度公式计算得出结果：1。也就是口味完全一致。显然与我们的认知是不一致的。
![](/media/15601420452086.jpg)

让我们利用Mean normalization来重新调整用户A、B的评分。

![](/media/15601432984898.jpg)
https://en.wikipedia.org/wiki/Feature_scaling

average(x) = 3
max(x) = 5
min(x) = 1

这样评分向量变化为：

1. 用户A对商品的评分：(-0.5, -0.5, -0.5, -0.5, -0.5)
1. 用户B对商品的评分：(0.5, 0.5, 0.5, 0.5, 0.5)

归一化后的向量之间余弦值为-1。品味完全不一致，与预期相符。


这里有一个假设，用户A、B之间的评分宽容度是一致的。所谓宽容度，有人看完电影觉得不错，会给5分，总体评分都很高，而有人评价比较刻薄，不错的电影最多4分，总体评分偏低。

所以一般推荐系统中对评分做归一化，一般是基于用户维度做归一化，而不是总体用户。

例如，在Andrew Ng的Machine Learning课上，采用的是每个用户评分减去每个用户自己的评分均值。
![](/media/15601439981816.jpg)



### 相似度计算优化



